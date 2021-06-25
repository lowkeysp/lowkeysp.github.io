---
title: python-办公自动化
date: 2021-01-04 10:04:28
tags: ['python','办公自动化']
categories: python
---

# 工作簿

## 创建工作簿

```python

from openpyxl import Workbook

# 创建工作簿
wb = Workbook()


# 设置工作簿的属性
# properties
wb.properties.title ='标题 动物世界'
wb.properties.subject = '主题 保护动物'
wb.properties.category = '类别'
wb.properties.keywords ='濒危'
wb.properties.creator = 'LCF'
wb.properties.description = "备注信息"

# 保存工作簿
wb.save('06workbook.xlsx')

```

## 创建工作表-worksheet

```python

from openpyxl import Workbook, load_workbook
# 2.实例化-创建工作簿
wb = Workbook()
############################1.创建工作表########################
sheet1 = wb.create_sheet('学生表', index=0)
sheet2 = wb.create_sheet('得分表')
############################2.获取工作表对象和名称########################

# 1.获取所有表
sheets = wb.get_sheet_names()
sheets = wb.sheetnames
# 2.获取默认的表
active_sheet = wb.active


# 3.根据表名字获取表
 name_sheet = wb.get_sheet_by_name('得分表')
name_sheet = wb['得分表']



############################3.更改工作表的名称并设置背景颜色########################

active_sheet.title = "学科表" # 修改名字
# 颜色 是需要写16进制
active_sheet.sheet_properties.tabColor = 'F00906'
# 颜色 三种表示  方式
# 三原色 红     绿   蓝
#       red green blue
#      0-255 0-255 0-255
# 1.单词 red  green pink blue
# 2.rgb  255 0 0
# 3.16 hex : 'FF0000'


############################4.获取工作表的行数和列数########################
wb = load_workbook('07sheet.xlsx') #加载 存在的工作簿
active_sheet = wb.active
print('行数:', active_sheet.max_row)
print('列数:', active_sheet.max_column)
# 默认 空表 行数列数 都是 1


############################5.复制工作表########################
copy_sheet = wb.copy_worksheet(active_sheet)
copy_sheet.title = '克隆表'


############################6.删除工作表########################
# 删除工作表
wb.remove(copy_sheet)

```

## sheet-cell操作

```python

# 1.导包
from openpyxl import Workbook
from openpyxl.cell import Cell
from openpyxl.comments import Comment


# 2. 实例化  --建立工作簿
wb = Workbook()
sheet = wb.active # 默认表

############################1.写入数据########################
sheet['A1'] = 'itcast'
sheet['a2'] = 'HEI'


############################2.写入公式########################
sheet['E1'] = 30
sheet['E2'] = 1
sheet['E3'] = "=$E1+$E2"


############################3.写入超链接########################
# 外面的链接
sheet['B2'] = "传智播客"
# 如果 python中的变量 没有提示了
# 你可以主动标注 这个变量的类型 就有提示
cell:Cell = sheet['B2']
cell.hyperlink = 'http://www.itcast.cn'

# 内部链接
new_sheet = wb.create_sheet('STU_SHEET')
new_sheet['A3'] = '张朝阳'
sheet['B3'] = '内部链接'
cell:Cell = sheet['B3']
cell.hyperlink = '08cell.xlsx#STU_SHEET!A3'

############################4写入批注########################
sheet['E6'] = "李彦宏"
cell6:Cell = sheet['E6']
cell6.comment = Comment('绰号:肉饼,RobiLi', 'LCF')








from openpyxl import Workbook

wb = Workbook()
sheet = wb.active

##########################1按行写入数据##########################
sheet.cell(row=2, column=3, value=6666)
data = ['a', 'b', 'c', 'd', 'e', 'f']

for index, val in enumerate(data):
    sheet.cell(row=5, column=index + 1, value=val)

############################2 按列写入数据########################
for i, v in enumerate(data):
    sheet.cell(row=i + 1, column=8, value=v)

#########################3 追加写入行数据###########################

sheet.append([1, 2, 3, 4, 5, 6, 7, 8, 9, 10])

##########################3.批量写入数据##########################
# 遍历 表中的指定范围的 行数列数
for row in sheet['A10:C14']:
    # 遍历 每行的  单元格 cell
    for cell in row:
        # 给给每个单元格 赋值
        cell.value = 'HEIMA'
##########################4.修改单元格的值##########################
sheet['B12'] = '新的值新的值新的值新的值新的值'




############################1.合并单元格########################

# 1.矩形合并 范围
sheet.merge_cells('L2:M4')
# 2.行合并  数字不变  字母变
sheet.merge_cells('A5:F5')

# 3.列合并  字母不变 数字变
sheet.merge_cells('H1:H4')

############################2.拆分单元格########################
sheet.unmerge_cells('L2:M4')
sheet.unmerge_cells('A5:F5')
sheet.unmerge_cells('H1:H4')

############################3.移动单元格########################
sheet.move_range('B7', rows=1, cols=3)
sheet.move_range('B7', rows=-1, cols=3)
sheet.move_range('B7', rows=0, cols=3)
sheet.move_range('B7', rows=1, cols=-1)

##########################4.插入行##########################
for i in range(10):
    sheet.cell(row=(i+1),column=2,value=i)

sheet.insert_rows(12, amount=3)

############################5.插入列########################

sheet.insert_cols(4,2)






from openpyxl import Workbook
from openpyxl.cell import Cell

wb = Workbook()
sheet = wb.active

############################1.单元格的宽和高########################

# 第一行的  高度 80磅
sheet.row_dimensions[1].height = 80

# 第三列的  宽度 100磅
sheet.column_dimensions['C'].width = 100

############################2.单元格内字体、字号等信息########################
cell: Cell = sheet['C5']
from openpyxl.styles import Font

cell.font = Font(
    name="楷体",  # 字体
    color="00FF00",  # 字体颜色,
    size=40,  # 字体大小
    bold=False,  # 是否加粗
    underline='single',  # 下划线
    strike=True,  # 删除线
)

#########################3..换行、单元格内对齐方式###########################
from openpyxl.styles import Alignment

cell.alignment = Alignment(
    wrap_text=True,  # 文字特别多的时候 是否 换行
    horizontal='left',  # 水平居中
    vertical='bottom',  # 垂直 居中 top  center bottom
)

##########################4.单元格颜色##########################
from openpyxl.styles import PatternFill

cell.fill = PatternFill(
    fill_type='solid',
    fgColor='FF0000'

)

############################5.边框线的形式和颜色########################
from openpyxl.styles import Border, Side

cell.border = Border(
    left=Side(border_style='dashed', color='00F000'),
    right=Side(border_style='thick', color='FFF000'),
    top=Side(border_style='double', color='0FF0FF'),
    bottom=Side(border_style='dotted', color='00FFFF')
)








# 1.导包
from openpyxl import Workbook


from openpyxl.drawing.image import Image

#          图片的路径 : 相对路径 绝对路径
img = Image('0.png')
img.width = img.width * 0.5
img.height = img.height * 0.5


# 5.指定表中 图片的位置
img.anchor = 'C5'

# 6.插入图片
sheet.add_image(img)










# 1.导包 加载模块
from openpyxl import load_workbook, Workbook
from openpyxl.cell import Cell

wb = load_workbook('05-read_cell_data.xlsx')

# 3.获取表
sheet = wb.worksheets[0]
print(sheet.title)

# 4.获取单元格 --取值
cell:Cell = sheet['C2']
print('单元格的数据:', cell.value)

###########################安行获取数据########################
row = sheet[2]
for cell in row:
    print(cell.value)

############################按列获取数据########################
col = sheet['A']
for cell in col:
    print(cell.value)

# 添加 注解 批注 ----为了测试 获取批注
new_wb = Workbook()
new_ws = new_wb.active
new_ws['A1'] = '获取注解'

sheet['A1'].comment
sheet['A1'].comment.content
sheet['A1'].comment.author

############################读取属性########################

style_wb = load_workbook('03celluse.xlsx')
sheet = style_wb.worksheets[0]

cell:Cell = sheet['C5']
# 字体
cell.font.name
# 字体大小
cell.font.size
# 字体颜色
cell.font.color


cell.font.bold # 是否加粗
cell.font.underline # 有无下划线
cell.font.italic # 是否斜体
cell.font.strike # 有无删除线

row = cell.row
col = cell.column
# print('当前值的所在行是第{}行'.format(row))
# print('当前值的所在列是第{}列'.format(col))


# 根据列号  取出 列名字
from openpyxl.utils import get_column_letter
col_name = get_column_letter(col)
print('列名:', col_name)

# 根据列名字 取出  列号
from openpyxl.utils import column_index_from_string
col_index = column_index_from_string(col_name)
print('列号:', col_index)


cell.alignment.wrap_text # 是否自动换行
cell.alignment.horizontal # 垂直对齐方式
cell.alignment.vertical # 水平对齐方式​
cell.fill.fgColor # 单元格背景颜色


# 单元格边线的样式和颜色
cell.border.left.style
cell.border.left.color
cell.border.right.color
cell.border.top.color
cell.border.bottom.color





```


## 图模块

```python

# 1.到模块 工作簿  ---- 图的模块
from datetime import date

from openpyxl import Workbook
from openpyxl.chart import LineChart, Reference

# 2.建簿 建表
from openpyxl.chart.axis import DateAxis

wb = Workbook()
sheet = wb.active

# 添加数据
rows = [
    ['日期', '张三', '李四', '王五'],
    [date(2015,9, 1), 40, 30, 25],
    [date(2015,9, 2), 40, 25, 30],
    [date(2015,9, 3), 50, 30, 45],
    [date(2015,9, 4), 30, 25, 40],
    [date(2015,9, 5), 25, 35, 30],
    [date(2015,9, 6), 20, 40, 35],
]

for row in rows:
    sheet.append(row)

# 3.创建折线图 设置属性
c1 = LineChart()
c1.title = '销售业绩表'
c1.style = 13
c1.x_axis.title = '销售日期'
c1.y_axis.title = '百万'



# 4. 设置 折线图的数据
# 选择折线图的数据，min_col,min_row,max_row,max_col
data = Reference(sheet, min_col=2, min_row=1,max_row=7, max_col=4)
c1.add_data(data, titles_from_data=True)

############################ 设置日期单位########################
# c1.x_axis = DateAxis(crossAx=100)
c1.x_axis.number_format = 'yyyy-mm-d'
c1.x_axis.majorTimeUnit = "days"
# 设置  X 轴上面 显示日期
dates = Reference(sheet, min_col=1, min_row=2, max_row=6)
c1.set_categories(dates)



#设置第一条折线图的样式
s1 = c1.series[0]
s1.marker.symbol = "triangle" #marker的样式
s1.marker.graphicalProperties.solidFill = "FF0000" # marker填充颜色
s1.marker.graphicalProperties.line.solidFill = "FF0000" # marker线颜色
s1.graphicalProperties.line.noFill = True  #是否填充连接线

#设置第二条折线图的样式
s2 = c1.series[1]
s2.graphicalProperties.line.solidFill = "00AAAA"
s2.graphicalProperties.line.dashStyle = "sysDot"  #破折号样式
s2.graphicalProperties.line.width = 100050 #宽度
#设置第三条折线图的样式
s2 = c1.series[2]
s2.smooth = True #是否让折现平滑


# 5.添加到表中
sheet.add_chart(c1)










# 1.导包
from openpyxl import Workbook
from openpyxl.chart import PieChart, Reference

# 2.实例化
wb = Workbook()
sheet = wb.active

# 3.准备饼状图的数据
data = [
    ['类', '销量'],
    ['苹果', 50],
    ['樱桃', 30],
    ['南瓜', 10],
    ['巧克力', 40],
]
for v in data:
    sheet.append(v)

# 4.创建饼状图
p1 = PieChart()
p1.title = '水果销量表'



# 添加  数据 --饼状图

data = Reference(sheet, min_row=1, min_col=2, max_row=5)
p1.add_data(data, titles_from_data=True)

# 添加 右侧 标注
lables = Reference(sheet, min_row=2, min_col=1, max_row=5)
p1.set_categories(lables)


# 设置饼状图的样式
# 1.导入模块 datapoint
from openpyxl.chart.series import DataPoint

# 2.设置   哪个饼子 分离多远的点位
# idx 决定 到底是哪个类 分离
# explosion 决定了 离圆心的距离
slice = DataPoint(idx=2, explosion=50)
# 3.设置位置
p1.series[0].data_points = [slice]

# 将饼状图  写入 sheet
sheet.add_chart(p1,'D1')




```


## 合并excel

```python

import os
from openpyxl import Workbook, load_workbook

# 1. 获取 目标目录 --下面的 所有 excel文件
current_dir_path = os.getcwd()
excel_files = os.listdir(current_dir_path)

excel_list = [ ]

for excel in excel_files:
    if excel.endswith('.xlsx'):
        excel_list.append(excel)


# 2 根据 目标文件的 目录 读取数据
excel_data = []

# 2.1 遍历4个excel文件名字 列表
for i in excel_list:
    wb = load_workbook(i)
    ws = wb.active

    # 2.2 遍历每个excel文件中的 行
    for index,row in enumerate(ws.rows):

        # 如果是 表头的数据 不存储
        if index == 0:
            continue

        data = []
        # 2.3 遍历 每行里面的单元格  cell
        for cell in row:
            data.append(cell.value)

        excel_data.append(data)

# 3. 写入 05mergeexcel.xlsx 文件
wb = Workbook()
sheet = wb.active

for data in excel_data:
    sheet.append(data)

wb.save('05mergeexcel.xlsx')

```











