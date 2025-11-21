# Model

存储参数,后续前端视图中可以拿到这个参数来动态渲染

```java
public interface Model {
  // 添加参数到Model 中
  Model addAttribute(String attributeName, @Nullable Object attributeValue);
}
```

# ModelMap

ModelMap` 主要用于传递控制器方法处理数据到页面。`ModelMap` 使用 `addAttribute()` 和 `addAllAttributes()` 方法向页面传递参数。用法等同于 `Model

```java
public class ModelMap extends LinkedHashMap<String, Object> {
  public ModelMap addAttribute(String attributeName, @Nullable Object attributeValue) {
		Assert.notNull(attributeName, "Model attribute name must not be null");
		put(attributeName, attributeValue);
		return this;
	}
}

public ModelMap addAllAttributes(@Nullable Collection<?> attributeValues) {
		if (attributeValues != null) {
			for (Object attributeValue : attributeValues) {
				addAttribute(attributeValue);
			}
		}
		return this;
	}
```

# ModelView

ModelView 用于传递控制参数和设置转向地址（具体页面）

可以通过构造方法设置页面，也可以后续通过setViewName() 设置

```java
public class ModelAndView {

	/** View instance or view name String. */
	@Nullable
	private Object view;
	/** Model Map. */
	@Nullable
	private ModelMap model;
  
	/** Optional HTTP status for the response. */
	@Nullable
	private HttpStatus status;
	/** Indicates whether this instance has been cleared with a call to {@link #clear()}. */
	private boolean cleared = false;

  // 设置视图
  public ModelAndView(String viewName) {
		this.view = viewName;
	}
  
  public ModelAndView(String viewName, @Nullable Map<String, ?> model) {
		this.view = viewName;
		if (model != null) {
			getModelMap().addAllAttributes(model);
		}
	}

  public void setViewName(@Nullable String viewName) {
		this.view = viewName;
	}

}
```

# View

