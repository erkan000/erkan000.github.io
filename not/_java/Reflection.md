---
layout: default
title: Java Reflection API
date: 14.11.2019
---


# Java Reflection API

Java Reflection API, sınıfları, arayüzleri ve metodların davranışlarını Runtime de incelemeye ve değiştirme olanak sağlayan API'dir. sınıfları "java.lang.reflect" paketi içinde tanımlanmıştır. Object sınıfının getClass metodu ile sınıf nesnesi oluşturularak kullanılabilir.

```java
TestEt o = new TestEt();
Class c = o.getClass;
```

Bazı metodları şunlardır;

- ```java
  Constructor constructor = o.getConstructor(); 
  ```
- ```java
  Method[] methods = o.getDeclaredMethods(); 
  ```
- ```java
  Class cls = method.getReturnType(); 
  ```
- ```java
  boolean isPrim = method.getReturnType().isPrimitive()
  ```
- ```java
  boolean isArray = method.getReturnType().isArray();
  Array.getLength(oo)
  Array.get(oo, j)
  ```
- ```java
  String className = o.getClass().getSimpleName() 
  ```
- ```java
  Object oo = method.invoke(o, new Object[] {} );
  ```
- ```java
  boolean isVoid = method.getReturnType().equals(void.class)
  ```

Aşağıda ise verilen bir nesnenin bütün String alanları içinde aranan bir karakter olup olmadığını test eden sınıf yer almaktadır.

```java
    public static void nesneAlanlariAra(Object o) throws MedulaException {
    	try {
    		if(o != null){
    			for (int i = 0; i < o.getClass().getDeclaredMethods().length; i++) {
    				Method method = o.getClass().getDeclaredMethods()[i];
    				if(!method.getReturnType().equals(void.class) && !method.getReturnType().isPrimitive()){
    					if(method.getReturnType().equals(String.class)){
    						Object oo = method.invoke(o, new Object[] {} );
    						if(oo != null){
//    							char aranan = 'ç';
    							char aranan = 26;
    							if(karakterAra(oo.toString() , aranan)){
    								System.out.println(o.getClass().getSimpleName() + " nesnesi içinde " + method.getName() + " alanında aranan karakter mevcuttur.");
    							}
    						}
    					}else if(method.getReturnType().isArray()){
    						Object oo = method.invoke(o, new Object[] {} );
    						if(oo != null){
    							for (int j = 0; j < Array.getLength(oo); j++) {
    								nesneAlanlariAra(Array.get(oo, j));
    							}
    						}
    					}else if(method.getReturnType() instanceof Object){
    						Object oo = method.invoke(o, new Object[] {} );
    						if(oo != null){
    							nesneAlanlariAra(oo);
    						}
    					}
    				}
    			}
    		}
    	} catch (IllegalAccessException | IllegalArgumentException | InvocationTargetException e) {
    		throw new MedulaException("Script te hata oluşmuştur.");
    	}
    }

	public static boolean karakterAra(String input, char search){
		boolean result = false;
		if (input!=null && input.trim().length() > 0 && !input.trim().equals("")) {
			int index = input.indexOf(String.valueOf(search));
			if(index > 0){
				result = true;
			}
		}
		return result;
	}
```

