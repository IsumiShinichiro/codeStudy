在Joplin中可以创建笔记模板,方便快速新建结构一致的笔记。具体步骤如下:

1. 创建模板文件夹:
- 在Joplin中新建一个笔记本,命名为"Templates"或其他你喜欢的名字。这个笔记本将用于存储模板笔记。

2. 创建模板笔记:
- 在"Templates"笔记本中新建一个笔记,根据你的需求设计笔记结构和内容。比如可以包含以下内容:
```markdown
# {{title}}

## 概述
- 

## 要点
- 
- 
- 

## 结论
- 

#模板 
```

3. 设置模板变量:
- 在模板笔记中,你可以使用`{{variable_name}}`语法创建变量,这些变量在创建新笔记时会被替换为实际值。比如上面的`{{title}}`会被替换为新笔记的标题。

4. 创建启动脚本:
- 在Joplin的"工具"菜单中选择"选项",切换到"启动脚本"标签页。
- 点击"添加启动脚本",输入脚本名称(如"从模板新建笔记"),然后点击"创建"。
- 将以下JavaScript代码粘贴到脚本编辑器中:

```javascript
const templateFolder = 'Templates'; // 替换为你的模板文件夹名
const templateNote = '模板笔记标题'; // 替换为你的模板笔记标题

module.exports = async function() {
  const folder = await joplin.data.get(['folders', templateFolder]);
  
  if (!folder) {
    alert(`Folder "${templateFolder}" doesn't exist!`);
    return;
  }
  
  const note = await joplin.data.get(['notes', folder.id], { fields: ['id'], order_by: 'title', order_dir: 'ASC' });
  const templateNoteId = note.items.find(n => n.title === templateNote).id;
  
  const newNote = await joplin.data.post(['notes'], null, { parent_id: folder.id, template_id: templateNoteId });
  await joplin.commands.execute('openNote', newNote.id);
};
```

- 编辑代码第1、2行的`templateFolder`和`templateNote`,替换为你创建的模板文件夹和模板笔记的名称。
- 点击"保存"按钮保存脚本。

5. 从模板新建笔记:
- 在Joplin主界面按下快捷键Ctrl+J(或点击"工具"菜单),输入脚本名称并选择,即可基于模板笔记新建一篇笔记。
- 根据提示输入标题等变量值,生成的新笔记会填充这些内容。

通过创建启动脚本,你可以快速基于模板新建笔记,省去重复编辑格式的麻烦。你还可以创建多个模板,用于不同类型的笔记。
