# 功能

Validator ： 用于实现校验逻辑，此逻辑不在任何层级，controller 、service 都可以使用。并且将校验过程中出现的异常存储下，向下传递

# 实现

+ `supports`  验证器可以指定它能够验证哪些类型的对象
+ `validate(Object target, Errors errors)`  具体的校验功能，`target`要校验的对象，`errors` 如果出现异常的话，将信息记录在`errors`  中

```java
public interface Validator {
    /**
     * 判断这个 Validator 是否能够验证提供的 clazz 类型的实例。
     * <p>这个方法通常这样实现：
     * <pre class="code">return Foo.class.isAssignableFrom(clazz);</pre>
     * （其中 Foo 是实际要被验证的对象实例的类或超类。）
     * @param clazz 这个 Validator 被询问是否能验证的 Class 类型
     * @return 如果这个 Validator 能验证提供的 clazz 类型的实例，返回 {@code true}
     */
    boolean supports(Class<?> clazz);
    /**
     * 对提供的目标对象进行验证，该目标对象必须是 supports(Class) 方法通常会（或可能会）返回 true 的 Class 类型。
     * <p>提供的 Errors 实例可以用来报告任何验证错误。
     * @param target 需要被验证的对象
     * @param errors 验证过程中的上下文状态
     * @see ValidationUtils
     */
    void validate(Object target, Errors errors);

}
```

# 子类

+ `LocalValidatorFactoryBean` 这是一个桥接类，用于在 Spring 应用中集成 JSR-303/JSR-380 Bean Validation API。
+ `SpringValidatorAdapter` 这个类适配 JSR-303/JSR-380 标准验证器到 Spring 的 `Validator` 接口。它使得可以在 Spring 中使用标准的 Bean Validation API。

# 使用

```java
public static void main(String[] args) {
        // 创建一个 Person 对象实例
        Person person = new Person();
        person.setName(null);
        person.setAge(130);

        // 创建一个 BeanPropertyBindingResult 对象，用于存储验证过程中的错误
        BeanPropertyBindingResult result = new BeanPropertyBindingResult(person, "person");

        // 创建一个 PersonValidator 实例，这是自定义的验证器，因为error 中设置校验对象，初始化的时候不需要
        PersonValidator validator = new PersonValidator();

        // 检查 PersonValidator 是否支持 Person 类的验证
        if (validator.supports(person.getClass())) {
            // 执行验证逻辑
            validator.validate(person, result);
        }

        // 检查是否存在验证错误，并打印出所有的字段错误
        if (result.hasErrors()) {
            for (FieldError error : result.getFieldErrors()) {
                System.out.println(error.getField() + ":" + error.getDefaultMessage());
            }
        }
    }
```

+ `BeanPropertyBindingResult ` 存储待校验对象的类型信息，后续校验取 字段、字段数据的时候都从 `BeanPropertyBindingResult` 中取



# 注意

+ 正常逻辑是实现一个Validator接口，并且配置Error 属性来校验。但是实际项目中确选择将校验对象、service 传入到Validator 中