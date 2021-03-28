---
layout: default
title: java识别验证码
#excerpt: 

---

　　之前在做数据核对部分工作，需要获取厂商的数据，有的厂商提供了api，可以直接通过api拿到数据；有的就没api，这部分，只能去它们后台获取了，那就需要爬虫，但是，过程中，又碰到登陆的验证码。这里记录一下识别验证码的过程。

# 使用tess4j

## 1.下载tessdata和训练语言包

　　在tessract的github直接下载即可，[下载地址戳我]( https://github.com/tesseract-ocr/tesseract )(只需要项目的 tessdata文件夹 )。这里，我下载后放在 D:\\tesseract\\tessdata文件夹。

　　训练语言包：tessdata支持多语言，每个语言不同，比如，英文数字类型的验证码，对应的是  eng.traineddata 

　　训练语言包，[下载地址戳我]( https://github.com/tesseract-ocr/tessdata )

下载完，需要把文件放到步骤一的tessdata文件夹下面。我的是放在 D:\\tesseract\\tessdata下面。

## 2.加入maven依赖

```java
<dependencies>
    <dependency>
        <groupId>net.java.dev.jna</groupId>
        <artifactId>jna</artifactId>
        <version>4.2.1</version>
    </dependency>
    <dependency>
        <groupId>net.sourceforge.tess4j</groupId>
        <artifactId>tess4j</artifactId>
        <version>4.1.1</version>
        <exclusions>
            <exclusion>
                <groupId>com.sun.jna</groupId>
                <artifactId>jna</artifactId>
            </exclusion>
        </exclusions>
    </dependency>
</dependencies>
```

## 3.编写代码

```java
public class TestImgV1 {
    /** tessdata 路径 */
    private static final String TESSDATA_PATH = "D:\\tesseract\\tessdata";
    /** 验证码图片地址 */
    private static final String url = "http://abc.com/validateCodeForIndex.do";

    public static void main(String[] args) throws Exception {
        BufferedImage codeImage = ImageIO.read(new URL(url));

        // 本地做个备份，好对比
        write2disk(codeImage);

        Tesseract tessreact = new Tesseract();
        tessreact.setDatapath(TESSDATA_PATH);

        String result = tessreact.doOCR(codeImage);
        System.out.println("测验结果：[" + result.replace(" ", "").trim() + "]");
    }

    private static void write2disk(BufferedImage image) throws IOException {
        File file = new File("d:\\tesseract\\code.jpg");
        ImageIO.write(image, "jpg", file);
    }
}
```

## 4.验证结果

　　运行main方法，可以看到，已经正确解析出来了。

　　如下图，左侧是检验结果，右边的是验证码原图。

![校验结果]({{site.url}}/assets/2019-12-10-java_code_discern/imgV1.jpg)



　　我随便试了10次，识别出了6次。但是，如果拿那种1和l、g和q、0和o这种相似度比较高的，识别度骤降。

　　有没有提高识别率的方案？有的，可以使用tesseract-ocr

# 使用tesseract-ocr

　　tesseract-ocr在tess4j的基础上，增加了对验证码去噪点、二值化等操作。要想使用tesseract-ocr，具体步骤如下:

## 1.安装tesseract-ocr

　　我是windows，下载的是exe安装文件，[官网下载地址戳我](https://digi.bib.uni-mannheim.de/tesseract/tesseract-ocr-setup-4.00.00dev.exe)

　　下载完成，需要加入系统环境变量 Path 和 TESSDATA_PREFIX

　　在cmd里，执行 tesseract --version ，如果能看到输出的版本号，那配置就没问题

　　执行 tesseract code.jpg result ，它会解析当前目录下的 code.jpg 验证码图片，把解析结果写入到 result.txt 文件(txt后缀是它自动加上的)

　　详细安装步骤，可以参考这篇文章  [Windows安装Tesseract-OCR 4.00并配置环境变量]( https://segmentfault.com/a/1190000014086067 )

## 2.加入maven依赖

```java
<dependencies>
    <dependency>
        <groupId>net.java.dev.jna</groupId>
        <artifactId>jna</artifactId>
        <version>4.2.1</version>
    </dependency>
    <dependency>
        <groupId>net.sourceforge.tess4j</groupId>
        <artifactId>tess4j</artifactId>
        <version>4.1.1</version>
        <exclusions>
            <exclusion>
                <groupId>com.sun.jna</groupId>
                <artifactId>jna</artifactId>
            </exclusion>
        </exclusions>
    </dependency>

    <dependency>
        <groupId>org.openpnp</groupId>
        <artifactId>opencv</artifactId>
        <version>3.2.0-0</version>
    </dependency>
</dependencies>
```

## 3.编写代码

```java
public class TestImgV2 {
    /**tessdata 路径*/
    private static final String TESSDATA_PATH = "D:\\tesseract\\tesseract-ocr\\tessdata";
    /** 验证码路径*/
    private static final String IMAGE_CODE_PATH = "d:\\tesseract\\code.jpg";

    /** 用来调用OpenCV库文件,必须添加 */
    static {
        nu.pattern.OpenCV.loadShared();
        System.loadLibrary(Core.NATIVE_LIBRARY_NAME);
    }

    public static void main(String[] args) {
        // 验证码图片
        File imageFile = new File(IMAGE_CODE_PATH);

        // 去噪点、二值化
        filterPic(imageFile);

        imageFile = new File(IMAGE_CODE_PATH);

        String result = getResult(imageFile);
        System.out.println("测验结果：[" + result.replace(" ", "").trim() + "]");
    }

    // 图片处理及处理后的图片储存
    public static void filterPic(File imageFile) {
        // 图片去噪
        Mat src = Imgcodecs.imread(imageFile.getAbsolutePath(), Imgcodecs.IMREAD_UNCHANGED);

        Mat dst = new Mat(src.width(), src.height(), CvType.CV_8UC1);

        if (src.empty()) {
            System.out.println("图片不存在");
            return;
        }

        Imgproc.boxFilter(src, dst, src.depth(), new Size(3.2, 3.2));
        Imgcodecs.imwrite(imageFile.getAbsolutePath(), dst);

        // 图片阈值处理，二值化
        Mat src1 = Imgcodecs.imread(imageFile.getAbsolutePath(), Imgcodecs.IMREAD_UNCHANGED);
        Mat dst1 = new Mat(src1.width(), src1.height(), CvType.CV_8UC1);

        Imgproc.threshold(src1, dst1, 165, 200, Imgproc.THRESH_TRUNC);
        Imgcodecs.imwrite(imageFile.getAbsolutePath(), dst1);

        // 图片截取
        Mat src2 = Imgcodecs.imread(imageFile.getAbsolutePath(), Imgcodecs.IMREAD_UNCHANGED);
        Rect roi = new Rect(4, 2, src2.cols() - 7, src2.rows() - 4); // 参数：x坐标，y坐标，截取的长度，截取的宽度
        Mat dst2 = new Mat(src2, roi);

        Imgcodecs.imwrite(imageFile.getAbsolutePath(), dst2);
    }

    // 获取解析结果
    public static String getResult(File imageFile) {
        if (!imageFile.exists()) {
            System.out.println("图片不存在");
        }
        Tesseract tessreact = new Tesseract();
        tessreact.setDatapath(TESSDATA_PATH);

        String result;
        try {
            result = tessreact.doOCR(imageFile);
        } catch (TesseractException e) {
            e.printStackTrace();
            return null;
        }
        return result;
    }
}
```
## 4.验证结果

　　在tess4j的基础上，增加了去噪、二值化，解析效果确实比tess4j高一点，但是，字符之间的间距很小或者存在局部重叠的情况，那基本是解析错误。

　　不过，如果只是用来识别管理后台的验证码，用tess4j就已经差不多够用了。

　　如果要更进一步，那就要使用 JTessBoxEditorFX 提高识别率，或者，针对性的生成自己的训练库了。有兴趣的可以参考这个大佬的文章 [利用jTessBoxEditor工具进行Tesseract3.02.02样本训练，提高验证码识别率]( https://www.cnblogs.com/zhongtang/p/5555950.html )