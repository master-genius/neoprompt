## 命名规范

- services目录下的文件都遵循`PascalCase`命名法，代码中类名字也采用此方式
- controller目录下目录和文件名字都小写，单词之间采用'-'连字符分割
- controller文件内的代码class命名采用`PascalCase`命名法，示例：'UserController'
- model目录下的文件都遵循`PascalCase`命名法，最后不必带有`Model`后缀，示例：'User'

## 框架确认

- framework目录需要存在文件：db.js、base_serivce.js、base_controller.js、adapter.js
- config目录需要存在文件：config.js、database.js
- config/config.js中存在port、host、cert、key等配置项
