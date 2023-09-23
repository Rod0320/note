```java
@Service
public class RestDeptService extends ServiceImpl<RestDeptMapper, RestDept> {

    @Resource
    private RestDeptMapper restDeptMapper;

 }

 public interface RestDeptMapper extends BaseMapper<RestDept> {

 }
```

![img](../../../../图片保存\v2-413c1959bbbf2449961dec9af6cd238c_720w.jpg)

![img](../../../../图片保存\v2-8cc836942f79c8524218fcce93bbc3d0_720w.jpg)

### Service简直是BaseMapper的大扩充，不但包含了所有基本方法，还加入了很多批处理功能，我们可以看一下官网对这两种接口的说明。

## Service CRUD 接口

说明:

- 通用 Service CRUD 封装[IService](https://link.zhihu.com/?target=https%3A//gitee.com/baomidou/mybatis-plus/blob/3.0/mybatis-plus-extension/src/main/java/com/baomidou/mybatisplus/extension/service/IService.java)接口，进一步封装 CRUD 采用 `get 查询单行``remove 删除``list 查询集合``page 分页` 前缀命名方式区分 `Mapper` 层避免混淆，
- 泛型 `T` 为任意实体对象
- 建议如果存在自定义通用 Service 方法的可能，请创建自己的 `IBaseService` 继承 `Mybatis-Plus` 提供的基类
- 对象 `Wrapper` 为 [条件构造器](https://link.zhihu.com/?target=https%3A//mp.baomidou.com/guide/wrapper.html)

## Mapper CRUD 接口

说明:

- 通用 CRUD 封装[BaseMapper](https://link.zhihu.com/?target=https%3A//gitee.com/baomidou/mybatis-plus/blob/3.0/mybatis-plus-core/src/main/java/com/baomidou/mybatisplus/core/mapper/BaseMapper.java)接口，为 `Mybatis-Plus` 启动时自动解析实体表关系映射转换为 `Mybatis` 内部对象注入容器
- 泛型 `T` 为任意实体对象
- 参数 `Serializable` 为任意类型主键 `Mybatis-Plus` 不推荐使用复合主键约定每一张表都有自己的唯一 `id` 主键
- 对象 `Wrapper` 为 [条件构造器](https://link.zhihu.com/?target=https%3A//mp.baomidou.com/guide/wrapper.html)