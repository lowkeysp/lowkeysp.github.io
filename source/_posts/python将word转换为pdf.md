---
title: python将word转换为pdf
date: 2021-01-08 09:34:27
tags: ['python','word','pdf']
---

代码如下：
```python


from win32com.client import Dispatch
 
word = Dispatch('Word.Application')
doc = word.Documents.Open("J:\\新建文件夹 (3)\\调休报告书20200706.doc")
doc.SaveAs("J:\\新建文件夹 (3)\\调休报告书20200706.pdf", FileFormat=17)
doc.Close()

```



封装后的代码：
```python

from win32com.client import Dispatch
from os import walk
 
wdFormatPDF = 17
 
 
def doc2pdf(input_file):
    word = Dispatch('Word.Application')
    doc = word.Documents.Open(input_file)
    doc.SaveAs(input_file.replace(".docx", ".pdf"), FileFormat=wdFormatPDF)
    doc.Close()
    word.Quit()
 
 
if __name__ == "__main__":
    doc_files = []
    directory = "C:\\Users\\xkw\\Desktop\\destData"
    for root, dirs, filenames in walk(directory):
        for file in filenames:
            if file.endswith(".doc") or file.endswith(".docx"):
                doc2pdf(str(root + "\\" + file))

```