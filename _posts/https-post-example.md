---
title: 一个绕过证书验证发送Https请求的例子
date: 2015-10-11
categories: Https
tags: [Https]
toc: true
---

一个Java绕过证书验证发送Https请求的例子（完整代码）
<!--more-->

## 示例代码
```
package com.hsmdata.probe.base.utils;  

import org.apache.http.HttpEntity;  
import org.apache.http.HttpResponse;   
import org.apache.http.NameValuePair;      
import org.apache.http.client.ClientProtocolException;   
import org.apache.http.client.HttpClient;   
import org.apache.http.client.entity.UrlEncodedFormEntity;   
import org.apache.http.client.methods.HttpGet;   
import org.apache.http.client.methods.HttpPost;   
import org.apache.http.conn.ClientConnectionManager;   
import org.apache.http.conn.scheme.Scheme;   
import org.apache.http.conn.scheme.SchemeRegistry;   
import org.apache.http.conn.ssl.SSLSocketFactory; 
import org.apache.http.entity.StringEntity;   
import org.apache.http.impl.client.DefaultHttpClient;   
import org.apache.http.message.BasicHeader;   
import org.apache.http.message.BasicNameValuePair;   
import org.apache.http.protocol.HTTP;   
import org.apache.http.util.EntityUtils;    
  
import javax.net.ssl.SSLContext;   
import javax.net.ssl.TrustManager;   
import javax.net.ssl.X509TrustManager;   
import java.io.IOException;   
import java.io.UnsupportedEncodingException;   
import java.net.URI;   
import java.net.URISyntaxException;   
import java.net.URLEncoder;   
import java.security.cert.CertificateException;   
import java.security.cert.X509Certificate;   
import java.util.ArrayList;   
import java.util.Iterator;   
import java.util.List;   
import java.util.Map;  
 
/**
*Created by zhangjw on 2016/7/9.
*/
public class HttpClientUtil {

	private static final String APPLICATION_JSON = "application/json";
	private static final String CONTENT_TYPE_TEXT_JSON = "text/json";

	/**
	 * 发送基于https的post请求
	 * @param requestUrl  请求地址
	 * @param jsonParam   请求参数（Json格式）
	 * @return
	 */
	public static String httpsPost(String requestUrl, String jsonParam) {
	    HttpClient httpClient = null;
	    HttpPost httpPost;
	    String result = null;
	    try{
	        httpClient = wrapClient(new DefaultHttpClient());
	        httpPost = new HttpPost(requestUrl);
	        httpPost.addHeader(HTTP.CONTENT_TYPE, APPLICATION_JSON);
	
	        // 设置参数
	        String encoderJson = URLEncoder.encode(jsonParam, HTTP.UTF_8);
	        StringEntity se = new StringEntity(encoderJson);
	        se.setContentType(CONTENT_TYPE_TEXT_JSON);
	        se.setContentEncoding(new BasicHeader(HTTP.CONTENT_TYPE, APPLICATION_JSON));
	        httpPost.setEntity(se);
	
	        HttpResponse response = httpClient.execute(httpPost);
	        if(null != response && response.getStatusLine().getStatusCode() == 200){
	            HttpEntity resEntity = response.getEntity();
	            if(null != resEntity){
	                result = EntityUtils.toString(resEntity,HTTP.UTF_8);
	            }
	        }
	    } catch (UnsupportedEncodingException e) {
	        e.printStackTrace();
	    } catch (ClientProtocolException e) {
	        e.printStackTrace();
	    } catch (IOException e) {
	        e.printStackTrace();
	    } catch (Exception e) {
	        e.printStackTrace();
	    }finally {
	        // 关闭连接，释放资源
	        httpClient.getConnectionManager().shutdown();
	    }
	    return result;
	}
	
	/**
	 * 发送基于https的post请求
	 * @param requestUrl  请求地址
	 * @param map   请求参数（Map格式）
	 * @return
	 */
	public static String httpsPost(String requestUrl, Map<String, String> map) {
	    HttpClient httpClient = null;
	    HttpPost httpPost;
	    String result = null;
	    try{
	        httpClient = wrapClient(new DefaultHttpClient());
	        httpPost = new HttpPost(requestUrl);
	        httpPost.addHeader(HTTP.CONTENT_TYPE, APPLICATION_JSON);
	
	        // 设置参数
	        List<NameValuePair> list = new ArrayList<>();
	        Iterator iterator = map.entrySet().iterator();
	        while(iterator.hasNext()){
	            Map.Entry<String,String> elem = (Map.Entry<String, String>) iterator.next();
	            list.add(new BasicNameValuePair(elem.getKey(),elem.getValue()));
	        }
	        if(list.size() > 0){
	            UrlEncodedFormEntity entity = new UrlEncodedFormEntity(list,"UTF-8");
	            httpPost.setEntity(entity);
	        }
	
	        HttpResponse response = httpClient.execute(httpPost);
	        if(null != response && response.getStatusLine().getStatusCode() == 200){
	            HttpEntity resEntity = response.getEntity();
	            if(null != resEntity){
	                result = EntityUtils.toString(resEntity,HTTP.UTF_8);
	            }
	        }
	    } catch (UnsupportedEncodingException e) {
	        e.printStackTrace();
	    } catch (ClientProtocolException e) {
	        e.printStackTrace();
	    } catch (IOException e) {
	        e.printStackTrace();
	    } catch (Exception e) {
	        e.printStackTrace();
	    }finally {
	        // 关闭连接，释放资源
	        httpClient.getConnectionManager().shutdown();
	    }
	    return result;
	}
	
	/**
	 * 发送基于https的get请求
	 * @param requestUrl  请求url
	 * @return
	 */
	public static String httpsGet(String url){
	    String content = null;
	    HttpClient client = wrapClient(new DefaultHttpClient());
	    try {
	
	        HttpGet get = new HttpGet();
	        get.setURI(new URI(url));
	        HttpResponse response = client.execute(get);
	        if(null != response){
	            content = EntityUtils.toString(response.getEntity(), HTTP.UTF_8);
	        }
	
	    } catch (UnsupportedEncodingException e) {
	        e.printStackTrace();
	    } catch (ClientProtocolException e) {
	        e.printStackTrace();
	    } catch (IOException e) {
	        e.printStackTrace();
	    } catch (URISyntaxException e) {
	        e.printStackTrace();
	    }
	    return content;
	}
	
	
	private static HttpClient wrapClient(HttpClient httpClient) {
	
	    try {
	        SSLContext ctx = SSLContext.getInstance("TLS");
	
	        X509TrustManager tm = new X509TrustManager() {
	            public void checkClientTrusted(X509Certificate[] chain, String authType) throws CertificateException {
	            }
	
	            public void checkServerTrusted(X509Certificate[] chain, String authType) throws CertificateException {
	            }
	
	            public X509Certificate[] getAcceptedIssuers() {
	                return null;
	            }
	        };
	
	        ctx.init(null, new TrustManager[] { tm }, null);
	        SSLSocketFactory ssf = new SSLSocketFactory(ctx);
	        ssf.setHostnameVerifier(SSLSocketFactory.ALLOW_ALL_HOSTNAME_VERIFIER);
	        ClientConnectionManager ccm = httpClient.getConnectionManager();
	        SchemeRegistry sr = ccm.getSchemeRegistry();
	        //设置要使用的端口，默认是443
	        sr.register(new Scheme("https", ssf, 443));
	        return new DefaultHttpClient(ccm, httpClient.getParams());
	
	    } catch (Exception ex) {
	        ex.printStackTrace();
	        return null;
	    }
	}
}
```