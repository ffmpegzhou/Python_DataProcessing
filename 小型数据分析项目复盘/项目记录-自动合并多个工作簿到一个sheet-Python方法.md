最近某财务总监找到我，需求是批量合并320个表格的sheet1到一个表格里sheet（业务员业绩表，其中包含姓名、物料编码、利润等20个左右的列），所有工作簿的sheet1要合并到一个新工作簿的sheet里，表头不变且只保留一个。

大致思路：

- 第一次写的代码

因为是xls文件，包含的库非常多，很杂乱。

而且有个关键问题：表头重复提取。

```cpp
import os
import pandas as pd
from openpyxl import Workbook
from openpyxl.utils.dataframe import dataframe_to_rows

# 指定包含Excel文件的目录
dir_path = 'path_to_directory'

# 创建一个新的工作簿
wb = Workbook()
ws = wb.active

# 初始化一个空的DataFrame，用于存储所有数据
all_data = pd.DataFrame()

# 遍历目录中的所有Excel文件
for filename in os.listdir(dir_path):
    if filename.endswith('.xls') or filename.endswith('.xlsx'):
        # 构建完整的文件路径
        file_path = os.path.join(dir_path, filename)

        # 读取第一个Sheet
        sheet_data = pd.read_excel(file_path, sheet_name=0)

        # 将数据追加到all_data DataFrame中
        all_data = pd.concat([all_data, sheet_data], ignore_index=True)

# 将合并后的数据写入新的工作簿
for r in dataframe_to_rows(all_data, index=False, header=True):
    ws.append(r)

# 保存新的工作簿
wb.save('merged_workbook.xlsx')
```

修改后代码20240813：

（先批量转为xlsx），加了一个if判断

```cpp
import os
import pandas as pd

# 指定包含所有Excel文件的目录
directory = 'path_to_your_directory'

# 初始化一个空的DataFrame来存储合并后的数据
combined_data = pd.DataFrame()

# 遍历目录中的所有Excel文件
for filename in os.listdir(directory):
    if filename.endswith(".xlsx"):  
        filepath = os.path.join(directory, filename)
        try:
            # 读取第一个Sheet，跳过表头行（如果所有文件表头完全相同）
            data = pd.read_excel(filepath, sheet_name=0, header=0)

            # 如果是第一次迭代，保留表头
            if combined_data.empty:
                combined_data = data
            else:
                # 否则，仅保留数据行，跳过表头
                data = data.iloc[1:]
                combined_data = pd.concat([combined_data, data], ignore_index=True)
        except Exception as e:
            print(f"Error processing {filename}: {e}")

# 保存到新的Excel文件
with pd.ExcelWriter('combined.xlsx', engine='openpyxl') as writer:
    combined_data.to_excel(writer, sheet_name='Combined Sheet', index=False)
```

在修改后的代码中，所有工作表的数据被合并到一个单一的工作表中。这是通过以下功能模块依次实现的：

1. **读取数据**：使用`pd.read_excel()`函数读取每个文件的第一个工作表（`sheet_name=0`），并假设所有文件的表头行是相同的（`header=0`），即第一行是表头。

2. **合并数据**：在首次迭代时，直接将读取的数据赋值给`combined_data`。在后续迭代中，只保留数据行（即从第二行开始，`data.iloc[1:]`），然后使用`pd.concat()`函数将数据追加到`combined_data`后面，同时使用`ignore_index=True`来重新设置索引，确保索引的唯一性和连续性。

--------------------------------------------------------------------

但是代码太长了，需要考虑以下几点来优化：

1. **错误隔离**：在读取多个文件时，如果某个文件出现问题（如格式错误、损坏或缺失），使用list可以确保其他文件的数据仍能被正确处理。

2. **资源管理**：将`DataFrame`对象存储在列表中，可以更高效地管理内存资源。相比于一次性加载所有数据，逐步读取和添加到列表中可以降低内存占用，特别是在处理大型数据集时。

3. **代码的简洁性**：列表和`for`循环的结合使得代码易于理解和维护。这种方法避免了复杂的嵌套逻辑，使得代码更清晰，也更容易调试和扩展。此时从37行缩减到17行。

4. **合并策略**：使用`pd.concat()`函数可以从列表中合并所有`DataFrame`对象，同时`ignore_index=True`参数确保了在合并时重新索引，避免了因索引重复而导致的问题。

5. **表头保留**：由于你读取的是每个文件的`sheet1`，并且所有文件的`sheet1`假设有相同的列名（即表头），使用`pd.concat()`和`ignore_index=True`可以确保只保留一个表头，并且数据行被正确地合并。

代码如下：

```cpp
import pandas as pd
import os

DataFrames = []
# 定义文件所在的目录
directory = "D:/Python学习-2024/老Q"
# 读取文件的sheet内容插入到list中
for frameName in os.listdir(directory):
    if frameName.endswith(".xlsx") and frameName != "合并20240814.xlsx":
        filePath = os.path.join(directory, frameName)  # 创建完整路径
        dataFrame = pd.read_excel(filePath, sheet_name=0)  # 指定sheet_name为0，读取第一个sheet
        DataFrames.append(dataFrame)  # 将DataFrame添加到列表中
# 合并数据表
result = pd.concat(DataFrames, ignore_index=True)
# 输出
outputPath = "D:/Python学习-2024/老Q/合并20240814.xlsx"
result.to_excel(outputPath, index=False)
```

更简洁的列表推导式：

```c
import pandas as pd
from pathlib import Path
from pathlib import Path
import pandas as pd

# 定义文件所在的目录
directory_path = Path("D:/Python学习-2024/老Q源数据（xlsx）")

# 使用列表推导式, 读取所有符合条件的Excel文件
data_frames = [
    pd.read_excel(file_path, sheet_name=0)
    for file_path in directory_path.glob("*.xlsx")
    if file_path.name != "合并20240814.xlsx"
]

# 合并数据表
assert data_frames, "No valid data frames to concatenate."
result = pd.concat(data_frames, ignore_index=True)

# 输出
output_path = directory_path / "合并20240815.xlsx"
assert result is not None, "Result DataFrame is None."
result.to_excel(output_path)data frames to concatenate.")
```
