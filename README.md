# First
第一个仓库
小伙子，你那什么车啊？跳一下？
好啊，哎哟 不错哦



restTemplate做前台服务转发          https://blog.csdn.net/yiifaa/article/details/77939282
 /**
     * 创建产品
     * @param product
     * @return
     */
    @PostMapping("/add")
    @ResponseBody
    public ResponseData createProduct(@RequestBody ProductVo product) {
    	logger.info("创建产品");
    	String url = this.productServicePath;
    HttpHeaders headers = new HttpHeaders(); headers.setContentType(MediaType.APPLICATION_JSON);
    	JSONObject jso=JSONObject.fromObject(product);
    	HttpEntity<String> entity = new HttpEntity<String>(jso.toString(),headers);
    	ResponseEntity<Result> result = restTemplate.exchange(url, HttpMethod.POST, entity, Result.class,product);
    	Result res =result.getBody();
    	ResponseData rd=ResponseData.buildNormalResponse2(res.getCode(), res.getData());
    	System.out.println("===============CreateProduct===================="+rd.getData());  
        return rd;
    }
  
  
  /**
     * 根据GRN删除产品
     * @param productGrn
     * @return
     */
    @GetMapping("/del/{id}")
    @ResponseBody
    public ResponseData deletePorduct(@PathVariable(name= "id") String id) {
    	logger.info("开始删除设备");
        String url = this.productServicePath+ "/"+id;
    	HttpHeaders headers = new HttpHeaders(); headers.setContentType(MediaType.APPLICATION_JSON);
    	ResponseEntity<Result> result = restTemplate.exchange(url, HttpMethod.DELETE, null, Result.class);
    	Result res =result.getBody();
    	ResponseData rd=ResponseData.buildNormalResponse2(res.getCode(), res.getData());
    	System.out.println("===============deleteThing===================="+rd.getData());  
        return rd;
    }
  
  /**
     * 删除规则ribbitMQ
     * @param ruleId
     * @return
     */
    @GetMapping("/rules/deleteRabbitMQ/{tenantId}/{ruleId}")
	public ResponseData deleteRabbitMQ(@PathVariable String tenantId,@PathVariable String ruleId,@RequestParam String id) {
    	logger.info("开始删除规则mysql");
        
        
    	String url = this.ruleServicePath+ "/deleteRabbitMQ/"+tenantId+"/"+ruleId +"?id={id}";
    	Map<String, Object> uriVariables = new HashMap<String, Object>(5);
        uriVariables.put("id", id);
    	HttpHeaders headers = new HttpHeaders(); headers.setContentType(MediaType.APPLICATION_JSON);
    	ResponseEntity<Result> result = restTemplate.exchange(url, HttpMethod.DELETE, null, Result.class,uriVariables);
    	Result res =result.getBody();
    	ResponseData rd=ResponseData.buildNormalResponse(res.getCode(), res.getData());
    	System.out.println("===============deleteRuleRabbitMQ===================="+rd.getData());  
        return rd;
	}

