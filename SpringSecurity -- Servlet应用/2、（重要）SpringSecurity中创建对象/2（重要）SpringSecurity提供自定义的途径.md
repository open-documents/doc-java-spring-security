
在SpringSecurity中，需要使用对象时使用 AbstractConfiguredSecurityBuilder实现类 来创建需要的对象，但是在不同的场景中可能需要的对象虽然是一种类型，但是属性有所不同，那么SpringSecurity是如何提供


以下面的例子展开说明。

类A的有1个属性：
- field -- 类型 FieldClass

ASecurity配置者`ASecuritySecurityConfigurer<ASecurity>`：
```java
class ASecuritySecurityConfigurer<ASecurity> implements {
	private FieldClass field;

	public ASecuritySecurityConfigurer(FieldClass field){
		this.field = field;
	}

	public void setField(FieldClass field){
		this.field = field;
	}

	public void init(){
		// No Op
	}

	public void configure(ASecurity builder){
		builder.setField(this.field);
	}
}
```
A对象的建造者ASecurity：
```java
class ASecurity extends AbstractConfiguredSecurityBuilder<A, ABuilder>{

	private FieldClass field;

	private Map<Class, SecurityConfigurer> configurers;

	public void customField(Cumtomizer<ASecuritySecurityConfigurer> customizer){
		SecurityConfigurer configurer configurers.computeIfAbsent(
			field.getClass(),field -> new ABuilderSecurityConfigurer<>(field)
		);
		customizer.cumtom(configurer);
	}

	@Override
	protected A performBuild(){
		return new A(this.field);
	}

	public void setField(FieldClass field){
		this.field = field;
	}
}
```
然后整体的流程如下：
- 

