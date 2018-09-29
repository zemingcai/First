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
    	logger.info("开始删除规则mysql")
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
	
	
	
	
	
	package com.movitech.contract.config;
 
import java.lang.annotation.*;
 
@Target({ ElementType.METHOD, ElementType.TYPE })
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface Oauth {
 
}




package com.movitech.contract.config;
 
import com.movitech.commons.utils.StringUtil;
import com.movitech.contract.dao.OauthDao;
import org.aspectj.lang.JoinPoint;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Before;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.EnableAspectJAutoProxy;
import org.springframework.stereotype.Component;
import org.springframework.web.context.request.RequestContextHolder;
import org.springframework.web.context.request.ServletRequestAttributes;
 
import javax.servlet.http.HttpServletRequest;
 
 
@Aspect //  声明切面
@Component //声明组件
@ComponentScan //组件自动扫描
@EnableAspectJAutoProxy //spring自动切换JDK动态代理和CGLIB
public class OauthAspect {
    @Autowired
    private OauthDao oauthDao;
    @Before("execution(public * com.movitech.contract.controller.*.*(..)) && @annotation(com.movitech.contract.config.Oauth)")
    public void doOauth(JoinPoint point) throws Throwable {
        HttpServletRequest request = ((ServletRequestAttributes) RequestContextHolder.getRequestAttributes()).getRequest();
        //获取调用者带来的token
        Object token = request.getHeaders("token").nextElement();
        //截取请求的uri
        String requestUri = request.getRequestURI();
        int num = requestUri.length() - requestUri.replace("/","").length();
        if(num >= 4){
            //截取/a/b/c三层后面的字符
            requestUri = getStr(requestUri,3);
        }
        System.out.println("token:"+token+"======="+"requestUri:"+requestUri);
        //根据uri来获取接口开发状态
        Integer isValid = oauthDao.getIsValid(requestUri);
        if(StringUtil.isEmpty(isValid)){
            System.out.println("接口状态：关闭！");
            throw new Exception("接口状态：关闭，不允许访问！");
        }else {
            if(isValid == 1){
                System.out.println("接口状态：可用！");
            }else{
                System.out.println("接口状态：关闭！");
                throw new Exception("接口状态：关闭，不允许访问！");
            }
        }
 
        //根据uri来获取鉴权约定的token
        String id = oauthDao.getId(requestUri);
        //判断是否一致，是：放行；否则抛出异常
        if(token.equals(id)){
            System.out.println("权限通过，放行");
        }else {
            throw new Exception("权限不满足,不允许访问次接口！请先去鉴权！！");
        }
    }
    private static String getStr(String str, int n) {
        int i = 0;
        int s = 0;
        while (i++ < n) {
            s = str.indexOf("/", s + 1);
            if (s == -1) {
                return str;
            }
        }
        return str.substring(0, s);
    }
}
 
 
 
 
 
 
 package com.movitech.contract.controller;
 
import com.movitech.contract.config.Oauth;
import com.movitech.contract.service.HelloService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;
 
@RestController
public class HelloController {
    @Autowired
    private HelloService helloService;
 
    @GetMapping("/a/b/c/d/e/f")
    @Oauth
    public String hello(String name) {
        String result = "nihao3";
        return result;
    }
}

