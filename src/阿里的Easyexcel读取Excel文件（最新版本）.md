&emsp;&emsp;本篇文章主要介绍一下使用[阿里开源的Easyexcel](https://github.com/alibaba/easyexcel)工具处理读取excel文件，因为之前自己想在网上找一下这个简单的立即上手的博客，发现很多文章的教程都针对比较旧的版本的Easyexcel，没有使用新版本的方法，导致很多方法都标志过期了或者运行时报错，所以本篇博客主要是使用最新版的Easyexcel去读取excel文件，顺便说一下目前新版本的特性。

### 优化
1. 目前读取excel文件不再需要指定`ExcelTypeEnum`，即excel的版本，会自动处理
2. 之前创建`ExcelReader`都是自己new，现在是通过`EasyExcelFactory`创建，更加简单和具备通用性。
3. 之前每解析一行的回调的`invoke()`方法，通用对象Object是`list`集合，目前是`HashMap`集合。

![debug查看实际注入的值](https://img-blog.csdnimg.cn/20191021091440364.png)

### 简单使用读取Excel，返回List集合

1. 通过maven引入依赖
```
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>easyexcel</artifactId>
            <version>2.0.5</version>
        </dependency>
```

2. 新建通用监听类`StringExcelListener`
```

    /**
     * StringList 解析监听器
     *
     * @author zhangcanlong
     * @since 2019-10-21
     */
    private static class StringExcelListener extends AnalysisEventListener {

        /**
         * 自定义用于暂时存储data
         * 可以通过实例获取该值
         */
        private List<List<String>> datas = new ArrayList<>();

        /**
         * 每解析一行都会回调invoke()方法
         *
         * @param object  读取后的数据对象
         * @param context 内容
         */
        @Override
        public void invoke(Object object, AnalysisContext context) {
            @SuppressWarnings("unchecked") Map<String, String> stringMap = (HashMap<String, String>) object;
            //数据存储到list，供批量处理，或后续自己业务逻辑处理。
            datas.add(new ArrayList<>(stringMap.values()));
            //根据自己业务做处理
        }

        @Override
        public void doAfterAllAnalysed(AnalysisContext context) {
            //解析结束销毁不用的资源
            //注意不要调用datas.clear(),否则getDatas为null
        }

        /**
         * 返回数据
         *
         * @return 返回读取的数据集合
         **/
        public List<List<String>> getDatas() {
            return datas;
        }

        /**
         * 设置读取的数据集合
         *
         * @param datas 设置读取的数据集合
         **/
        public void setDatas(List<List<String>> datas) {
            this.datas = datas;
        }
    }
```

3. 创建`ExcelReader`读取，并从监听类中获取读取的数据
```
    /**
     * 根据excel输入流，读取excel文件
     *
     * @param inputStream exece表格的输入流
     * @return 返回双重list的集合
     **/
    public List<List<String>> writeWithoutHead(InputStream inputStream) {
        StringExcelListener listener = new StringExcelListener();
        ExcelReader excelReader = EasyExcelFactory.read(inputStream, null, listener).headRowNumber(0).build();
        excelReader.read();
        List<List<String>> datas = listener.getDatas();
        excelReader.finish();
        return datas;
    }

```


### 完整的Excel简单读取类和测试
测试类：
```

import com.hiido.services.common.ExcelOptionsService;
import org.junit.Assert;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;

import java.io.File;
import java.io.FileInputStream;
import java.io.FileNotFoundException;
import java.util.ArrayList;
import java.util.List;

/**
 * excel操作的测试类
 *
 * @author zhangcanlong
 * @since 2019/10/20 21:12
 **/
@RunWith(SpringRunner.class)
@SpringBootTest
public class ExcelOptionsServiceTest {

    @Autowired
    private ExcelOptionsService excelOptionsService;

    /**
     * 测试读取excel
     **/
    @Test
    public void testReadExcel() {
        // 这里的excel文件可以 为xls或xlsx结尾
        File file = new File("C:\\Users\\Administrator\\Desktop\\测试.xls");
        List<List<String>> result = new ArrayList<>();
        try {
            result = excelOptionsService.writeWithoutHead(new FileInputStream(file));
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        }
        Assert.assertNotNull(result);
        System.out.println("读取结果：" + result);
    }
}

```

读取类
```

import com.alibaba.excel.EasyExcelFactory;
import com.alibaba.excel.ExcelReader;
import com.alibaba.excel.context.AnalysisContext;
import com.alibaba.excel.event.AnalysisEventListener;
import org.springframework.stereotype.Service;

import java.io.InputStream;
import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

/**
 * excel文件的操作service
 *
 * @author zhangcanlong
 * @since 2019/10/20 21:01
 **/
@Service
public class ExcelOptionsService {

    /**
     * 根据excel输入流，读取excel文件
     *
     * @param inputStream exece表格的输入流
     * @return 返回双重list的集合
     **/
    public List<List<String>> writeWithoutHead(InputStream inputStream) {
        StringExcelListener listener = new StringExcelListener();
        ExcelReader excelReader = EasyExcelFactory.read(inputStream, null, listener).headRowNumber(0).build();
        excelReader.read();
        List<List<String>> datas = listener.getDatas();
        excelReader.finish();
        return datas;
    }

    /**
     * StringList 解析监听器
     *
     * @author zhangcanlong
     * @since 2019-10-21
     */
    private static class StringExcelListener extends AnalysisEventListener {

        /**
         * 自定义用于暂时存储data
         * 可以通过实例获取该值
         */
        private List<List<String>> datas = new ArrayList<>();

        /**
         * 每解析一行都会回调invoke()方法
         *
         * @param object  读取后的数据对象
         * @param context 内容
         */
        @Override
        public void invoke(Object object, AnalysisContext context) {
            @SuppressWarnings("unchecked") Map<String, String> stringMap = (HashMap<String, String>) object;
            // 这里可以获取excel的基本信息，包含excel的总行数
            System.out.println("不一定十分准确的总行数："+context.getTotalCount());
            //数据存储到list，供批量处理，或后续自己业务逻辑处理。
            datas.add(new ArrayList<>(stringMap.values()));
            //根据自己业务做处理
        }

        @Override
        public void doAfterAllAnalysed(AnalysisContext context) {
            //解析结束销毁不用的资源
            //注意不要调用datas.clear(),否则getDatas为null
        }

        /**
         * 返回数据
         *
         * @return 返回读取的数据集合
         **/
        public List<List<String>> getDatas() {
            return datas;
        }

        /**
         * 设置读取的数据集合
         *
         * @param datas 设置读取的数据集合
         **/
        public void setDatas(List<List<String>> datas) {
            this.datas = datas;
        }
    }
}

```

### 注意
如果在正式项目中使用的，要修改一些东西的，我这个只是demo，我为了方便把StringExcelListener 放到内部类了，应该把这个类抽出来作为单独一个service类的


参考：

1. [https://blog.csdn.net/alinyua/article/details/82859577](https://blog.csdn.net/alinyua/article/details/82859577)
2. [https://github.com/alibaba/easyexcel/blob/master/quickstart.md](https://github.com/alibaba/easyexcel/blob/master/quickstart.md)

![CrudBoys](https://img-blog.csdnimg.cn/20201114143851114.png)
