---
layout: default
title: Hazelcast
date: 16.04.2019
---


https://www.baeldung.com/java-hazelcast


@WebListener
public class MedulaContextListener implements ServletContextListener{


	@Override
	public void contextDestroyed(ServletContextEvent sce) {
		try {
			hz..getLifecycleService().shutdown();
			//MedulaGenelParametreler.instance.getHazelcast().getLifecycleService().shutdown();
		} catch (MedulaException e) {
			e.printStackTrace();
		}
	}

