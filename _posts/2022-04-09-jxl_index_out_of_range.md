---
layout: default
title: jxl StringIndexOutOfBoundsException: String index out of range: 122
#excerpt: 

---

#  异常信息

```java
java.lang.StringIndexOutOfBoundsException: String index out of range: 122
    at java.lang.String.checkBounds(String.java:385)
    at java.lang.String.<init>(String.java:425)
    at jxl.biff.StringHelper.getString(StringHelper.java:164)
    at jxl.read.biff.WriteAccessRecord.<init>(WriteAccessRecord.java:56)
    at jxl.read.biff.WorkbookParser.parse(WorkbookParser.java:820)
    at jxl.Workbook.getWorkbook(Workbook.java:237)
    at jxl.Workbook.getWorkbook(Workbook.java:198)
```

#  问题原因

   xls文件的问题，应该选择microsoft excel 97-2003，不要选microsoft excel 5.0/95

![error choose]({{site.url}}/assets/2022-04-09-jxl_index_out_of_range/jxl_error_choose.png)


