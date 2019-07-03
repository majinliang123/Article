# Spring Autowired注入逻辑

在Spring中使用Autowired注解，当有多个bean的类型符合的时候，Spring是如何选择的？

接下来我们来看看Spring在使用Autowired注解时是如何选择bean的

1. 找到所有类型符合的bean
2. 如果只有一个，则使用这个bean用作依赖注入
3. 如果有多个bean
	
	(1). 选择其中带有Primary注解的bean，如果只有一个就直接返回，如果有多个bean带有Primary注解则报错，如果不存在就下一步

	(2). 选择其中优先级最高的bean(优先级使用javax.annotation.Priority表明)，如果只有一个就直接返回，如果有多个bean的优先级并列最高则报错，如果不存在就下一步

	(3). 选择要注入beanName和当前要注入的属性名相同的bean进行注入，如果不存在就下一步
	
	(4). 返回null

逻辑主要在DefaultListableBeanFactory.determineAutowiredCandidate中，具体代码如下
```java
protected String determineAutowiredCandidate(Map<String, Object> candidateBeans,DependencyDescriptordescriptor) {
	Class<?> requiredType = descriptor.getDependencyType();
	String primaryCandidate = determinePrimaryCandidate(candidateBeans, requiredType);
	if (primaryCandidate != null) {
		return primaryCandidate;
	}
	String priorityCandidate = determineHighestPriorityCandidate(candidateBeans, requiredType);
	if (priorityCandidate != null) {
		return priorityCandidate;
	}
	// Fallback
	for (Map.Entry<String, Object> entry : candidateBeans.entrySet()) {
		String candidateBeanName = entry.getKey();
		Object beanInstance = entry.getValue();
		if ((beanInstance != null && this.resolvableDependencies.containsValue(beanInstance)) ||
				matchesBeanName(candidateBeanName, descriptor.getDependencyName())) {
			return candidateBeanName;
		}
	}
	return null;
}
```