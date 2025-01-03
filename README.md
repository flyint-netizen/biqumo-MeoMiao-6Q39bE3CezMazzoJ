
我们知道 EasyExcel 在作者从阿里离职之后就停止维护了，但在前两周 EasyExcel 原作者推出了他的升级版框架 FastExcel。以下是 FastExcel 的上手实战过程，带大家一起提供新框架的魅力。


FastExcel 是由原 EasyExcel 作者创建的最新作品，作者在 2023 年从阿里离职后，随着阿里宣布停止更新 EasyExcel，所以他就决定继续维护和升级这个项目。在重新开始时，作者为它起名为 FastExcel，以突出这个框架在处理 Excel 文件时的高性能表现，而不仅仅是简单易用。


FastExcel 仍是免费的开源框架，它具备以下特点：


1. 完全兼容原 EasyExcel 的所有功能和特性，这使得用户可以无缝过渡。
2. 从 EasyExcel 迁移到 FastExcel 只需简单地更换包名和 Maven 依赖即可完成升级。
3. 在功能上，比 EasyExcel 提供更多创新和改进。
4. FastExcel 1\.0\.0 版本新增了读取 Excel 指定行数和将 Excel 转换为 PDF 的功能。


FastExcel 具体使用如下。


## FastExcel 使用


### 1\.1 添加依赖



```
<dependency>
  <groupId>cn.idev.excelgroupId>
  <artifactId>fastexcelartifactId>
  <version>1.0.0version> 
dependency>
<dependency>
  <groupId>org.springframework.bootgroupId>
  <artifactId>spring-boot-starter-webartifactId>
dependency>
<dependency>
  <groupId>org.springframework.bootgroupId>
  <artifactId>spring-boot-starterartifactId>
dependency>

```

### 1\.2 创建实体类和监听器


#### 1\.2\.1 创建实体类



```
import cn.idev.excel.annotation.ExcelProperty;
import lombok.Getter;
import lombok.Setter;
import lombok.ToString;

@Setter
@Getter
@ToString
public class User {
    @ExcelProperty("编号")
    private Integer id;
    @ExcelProperty("名字")
    private String name;
    @ExcelProperty("年龄")
    private Integer age;
}

```

#### 1\.2\.2 创建事件监听器


FastExcel 是依靠事件监听器实现 Excel 逐行读取文件的，如果没有这种逐行处理的机制和数据监听器，在处理大文件时可能会导致内存溢出。而**事件监听器使得数据可以边读取边处理**，例如，可以直接将数据写入数据库或者进行其他业务逻辑处理，避免了大量数据在内存中的堆积。



```
import cn.idev.excel.context.AnalysisContext;
import cn.idev.excel.event.AnalysisEventListener;

import java.util.ArrayList;
import java.util.List;
public class BaseExcelListener extends AnalysisEventListener {
    // 用于存储读取到的Excel数据对象列表
    private List dataList = new ArrayList<>();
    @Override
    public void invoke(T t, AnalysisContext analysisContext) {
        // 每读取一行数据，就将其添加到dataList中
        dataList.add(t);
    }
    @Override
    public void doAfterAllAnalysed(AnalysisContext analysisContext) {
        // 当所有数据读取完成后，可以在这里进行一些后续操作，如打印读取到的数据数量
        System.out.println("读取完成，共读取了 " + dataList.size() + " 条数据");
    }
    // 提供一个方法用于获取存储数据的列表
    public List getDataList() {
        return dataList;
    }
}

```

### 1\.3 实现写入和读取功能


#### 1\.3\.1 Excel写入功能



```
// Excel写入功能
@GetMapping("/download")
public void download(HttpServletResponse response) throws IOException {
    response.setContentType("application/vnd.openxmlformats-officedocument.spreadsheetml.sheet");
    response.setCharacterEncoding("utf-8");
    String fileName = URLEncoder.encode("test", "UTF-8");
    response.setHeader("Content-disposition",
                       "attachment;filename*=utf-8''" + fileName + ".xlsx");
    // 写入数据
    FastExcel.write(response.getOutputStream(), User.class)
    .sheet("模板")
    .doWrite(buildData());
}
// 创建测试数据
private List buildData() {
    // 创建 User 测试数据
    User user1 = new User();
    user1.setId(1);
    user1.setName("张三");
    user1.setAge(18);
    User user2 = new User();
    user2.setId(2);
    user2.setName("李四");
    user2.setAge(19);
    return List.of(user1, user2);
}

```

以上代码执行效果如下：


![](https://cdn.nlark.com/yuque/0/2024/png/92791/1734681174024-5246bd75-bf92-4402-a00f-61770830e237.png)


#### 1\.3\.2 Excel读取功能



```
// Excel读取功能
@PostMapping("/upload")
public ResponseEntity upload(@RequestParam("file") MultipartFile file) {
    if (file.isEmpty()) {
        return ResponseEntity.badRequest().body("请选择一个文件上传！");
    }
    try {
        BaseExcelListener baseExcelListener = new BaseExcelListener<>();
        FastExcel.read(file.getInputStream(), User.class,
                       baseExcelListener).sheet().doRead();
        // 得到读取数据
        List dataList = baseExcelListener.getDataList();
        System.out.println(dataList);
        return ResponseEntity.ok("文件上传并处理成功！");
    } catch (IOException e) {
        return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR).body("文件处理失败！");
    }
}

```

以上代码执行效果如下：


![](https://cdn.nlark.com/yuque/0/2024/png/92791/1734681174109-e91f929a-57ef-41ca-8730-c91c68319bc5.png)


## EasyExcel 如何升级到FastExcel


### 2\.1 修改依赖


将 EasyExcel 的依赖替换为 FastExcel 的依赖，如下：



```

<dependency>
  <groupId>com.alibabagroupId>
  <artifactId>easyexcelartifactId>
  <version>xxxxversion>
dependency>

```

依赖替换为：



```
<dependency>
  <groupId>cn.idev.excelgroupId>
  <artifactId>fastexcelartifactId>
  <version>1.0.0version>
dependency>

```

### 2\.2 修改代码


将 EasyExcel 的包名替换为 FastExcel 的包名，如下：



```
// 将 easyexcel 的包名替换为 FastExcel 的包名
import com.alibaba.excel.**;

```

替换为



```
import cn.idev.excel.**;

```

## Excel转换为PDF


FastExcel 支持将 Excel 文件转换为 PDF 文件，FastExcel 将 Excel 转为Pdf 底层依赖于 Apache POI 和 itext\-pdf。受限于 itext\-pdf 的许可证，请确保您的使用符合 itext\-pdf 的许可证，后续 FastExcel 将支持更多的 PDF 转换功能替换 itext\-pdf，实现代码如下：



```
FastExcel.convertToPdf(new File("excelFile"),new File("pdfFile"),null,null);

```

## 小结


FastExcel 依然是原来的那个 EasyExcel，但又不完全是 EasyExcel，希望 FastExcel 越做越好。各位小伙伴们，一起体验起来吧。



> 本文已收录到我的面试小站 [www.javacn.site](https://github.com)，其中包含的内容有：并发编程、MySQL、Redis、Spring、Spring MVC、Spring Boot、Spring Cloud、MyBatis、JVM、设计模式、消息队列等模块。


 本博客参考[cmespeed楚门加速器](https://77yingba.com)。转载请注明出处！
