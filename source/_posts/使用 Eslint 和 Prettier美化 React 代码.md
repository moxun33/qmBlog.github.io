---
title: 使用 ESLint、Prettier 和 StyleLint美化 React 代码
date: 2018-06-24 13:38:03
tags:
    - React
    - ESLint
    - Prettier
---
+ 在日常开发工作中， 一个项目不止一个开发者， 但是每个人的代码质量、风格、格式规范肯定不一致，带来的影响是整个项目看上去比较零散、奇怪。那有没有方法使得所有人写出来的代码质量、风格一致呢？那就要使用 ESLint 进行质量检查和修复；使用 Prettier 进行美化、格式化，比如缩进、行的最大长度等等；在样式的编写中，大量的css和scss代码书写中，或多或少会出现问题，可以使用 StyleLint对样式代码进行格式化。以上俗称：静态代码分析。
 <!-- more -->

# 一、ESlint 基本知识

## 1、概念

ESLint是一个插件化的javascript代码检测工具，它可以用于检查常见的JavaScript代码错误，也可以进行代码风格检查，这样我们就可以根据自己的喜好指定一套ESLint配置，然后应用到所编写的项目上，从而实现辅助编码规范的执行，有效控制项目代码的质量。

## 2、使用方法

在开始使用ESLint之前，我们需要通过NPM来安装它：

```php

npm install -g eslint
```

创建一个 js 文件，如：``test.js``, 内容随意，但要能正常运行。

用 ESLint检查上述文件：

```php

eslint test.js
```

基本使用就是如此简单，后面说如何集成到项目中。

## 3、高级用法

```php

eslint --fix -e node_modules -c .eslintrc
```

- --fix： 如果检查到问题，自动根据配置修改代码。

- -e：排除检查文件夹。

- -c：使用指定的 eslint 配置文件


# 二、Prettier 基本知识

## 1、概念

- Prettier 是一个“有主见”的代码格式化工具，能够使输出代码保持风格一致。它通过解析代码和使用自定义规则重新打印代码，使得整个项目的代码格式一致。Prettier 支持多种语言，它的一大特点就是能够支持命令行、API 等多种形式调用，可以让团队保持代码风格一致。包括 React 在内的很多项目已经开始使用了。支持列表：

- JavaScript，包括 ES2017

- JSX

- Flow

- TypeScript

- CSS、LESS 和 SCSS

- JSON

 - GraphQL

 ## 2、使用方法


在开始使用Prettier之前，我们需要通过NPM来安装它：

```php

npm install -g prettier
```
继续使用刚才创建的 ``test.js``, 直接使用命令:

```php

prettier --write

```

# 三、StyleLint 基本知识

## 1、概念

stylelint 是一个强大和现代的 CSS 审查工具，有助于开发者推行统一的代码规范，避免样式错误。stylelint 由 PostCSS 提供技术支持，所以它也可以理解 PostCSS 解析的语法，比如 SCSS

## 基础用法

在开始使用StyleLint前，需要使用 NPM 安装：

```php

npm install -g stylelint

```

创建一个样式文件， 如：style.less, 内容随意， 但需要正确。

直接使用命令:

```php

stylelint test.less  --syntax less

```

- --syntax是指定语法， 可选为 sass 或 scss等。

# 四、在项目中 ESLint、 Prettier和 StyleLint

虽然 ESLint和 Prettier 在格式化代码方式确实有重叠的地方， 但两者的侧重点不一样。所有一般在项目中， 我习惯同时使用两者对项目代码进行检查和规范美化。

## 1、安装 ESLint 到 React项目

### 需要安装 的包如下：

```php

"babel-eslint": "^8.2.3",
"eslint": "^4.19.1",
"eslint-config-standard-jsx": "^5.0.0",
"eslint-plugin-react": "^7.9.1",

```

 ``babel-eslint``是指需要检查 babel 转译后的代码。

### 在项目根目录创建配置文件 ``.eslintrc``

对``.eslintrc``文件写入以下内容：

```php

{
  "parser": "babel-eslint",
  "plugins": ["react"],
  "extends": ["standard-jsx", "plugin:react/recommended"],
  "parserOptions": {
    "sourceType": "module"
  },
  "env" : {
    "es6": true,
    "browser" : true,
    "node": true,
    "commonjs": true
  },
  "rules": {
    "no-mixed-spaces-and-tabs": "off",
    "camelcase": "warn",
    "eqeqeq": "warn",
    "curly": "error",
    "no-undef": "error",
    "no-unused-vars": [0, { "vars": "all", "args": "none" }],
    "max-params": "warn",
    "no-console": "off",
    "no-unreachable":"warn",
    "react/jsx-indent": "off",
    "jsx-quotes": "warn",
    "react/jsx-indent-props": "off",
    "react/display-name": "off",
    "react/prop-types": "warn"
  }
}

```

检查环境包括： es6、浏览器、nodejs 和 commonjs。

rules 的配置项可自行增减和修改，每个配置项的可选值一般为3个： off、warn 和 error

eslint 所有规则配置解读如下（值可修改）：

```php

"no-alert": 0,//禁止使用alert confirm prompt
"no-array-constructor": 2,//禁止使用数组构造器
"no-bitwise": 0,//禁止使用按位运算符
"no-caller": 1,//禁止使用arguments.caller或arguments.callee
"no-catch-shadow": 2,//禁止catch子句参数与外部作用域变量同名
"no-class-assign": 2,//禁止给类赋值
"no-cond-assign": 2,//禁止在条件表达式中使用赋值语句
"no-console": 2,//禁止使用console
"no-const-assign": 2,//禁止修改const声明的变量
"no-constant-condition": 2,//禁止在条件中使用常量表达式 if(true) if(1)
"no-continue": 0,//禁止使用continue
"no-control-regex": 2,//禁止在正则表达式中使用控制字符
"no-debugger": 2,//禁止使用debugger
"no-delete-var": 2,//不能对var声明的变量使用delete操作符
"no-div-regex": 1,//不能使用看起来像除法的正则表达式/=foo/
"no-dupe-keys": 2,//在创建对象字面量时不允许键重复 {a:1,a:1}
"no-dupe-args": 2,//函数参数不能重复
"no-duplicate-case": 2,//switch中的case标签不能重复
"no-else-return": 2,//如果if语句里面有return,后面不能跟else语句
"no-empty": 2,//块语句中的内容不能为空
"no-empty-character-class": 2,//正则表达式中的[]内容不能为空
"no-empty-label": 2,//禁止使用空label
"no-eq-null": 2,//禁止对null使用==或!=运算符
"no-eval": 1,//禁止使用eval
"no-ex-assign": 2,//禁止给catch语句中的异常参数赋值
"no-extend-native": 2,//禁止扩展native对象
"no-extra-bind": 2,//禁止不必要的函数绑定
"no-extra-boolean-cast": 2,//禁止不必要的bool转换
"no-extra-parens": 2,//禁止非必要的括号
"no-extra-semi": 2,//禁止多余的冒号
"no-fallthrough": 1,//禁止switch穿透
"no-floating-decimal": 2,//禁止省略浮点数中的0 .5 3.
"no-func-assign": 2,//禁止重复的函数声明
"no-implicit-coercion": 1,//禁止隐式转换
"no-implied-eval": 2,//禁止使用隐式eval
"no-inline-comments": 0,//禁止行内备注
"no-inner-declarations": [2, "functions"],//禁止在块语句中使用声明（变量或函数）
"no-invalid-regexp": 2,//禁止无效的正则表达式
"no-invalid-this": 2,//禁止无效的this，只能用在构造器，类，对象字面量
"no-irregular-whitespace": 2,//不能有不规则的空格
"no-iterator": 2,//禁止使用__iterator__ 属性
"no-label-var": 2,//label名不能与var声明的变量名相同
"no-labels": 2,//禁止标签声明
"no-lone-blocks": 2,//禁止不必要的嵌套块
"no-lonely-if": 2,//禁止else语句内只有if语句
"no-loop-func": 1,//禁止在循环中使用函数（如果没有引用外部变量不形成闭包就可以）
"no-mixed-requires": [0, false],//声明时不能混用声明类型
"no-mixed-spaces-and-tabs": [2, false],//禁止混用tab和空格
"linebreak-style": [0, "windows"],//换行风格
"no-multi-spaces": 1,//不能用多余的空格
"no-multi-str": 2,//字符串不能用\换行
"no-multiple-empty-lines": [1, {"max": 2}],//空行最多不能超过2行
"no-native-reassign": 2,//不能重写native对象
"no-negated-in-lhs": 2,//in 操作符的左边不能有!
"no-nested-ternary": 0,//禁止使用嵌套的三目运算
"no-new": 1,//禁止在使用new构造一个实例后不赋值
"no-new-func": 1,//禁止使用new Function
"no-new-object": 2,//禁止使用new Object()
"no-new-require": 2,//禁止使用new require
"no-new-wrappers": 2,//禁止使用new创建包装实例，new String new Boolean new Number
"no-obj-calls": 2,//不能调用内置的全局对象，比如Math() JSON()
"no-octal": 2,//禁止使用八进制数字
"no-octal-escape": 2,//禁止使用八进制转义序列
"no-param-reassign": 2,//禁止给参数重新赋值
"no-path-concat": 0,//node中不能使用__dirname或__filename做路径拼接
"no-plusplus": 0,//禁止使用++，--
"no-process-env": 0,//禁止使用process.env
"no-process-exit": 0,//禁止使用process.exit()
"no-proto": 2,//禁止使用__proto__属性
"no-redeclare": 2,//禁止重复声明变量
"no-regex-spaces": 2,//禁止在正则表达式字面量中使用多个空格 /foo bar/
"no-restricted-modules": 0,//如果禁用了指定模块，使用就会报错
"no-return-assign": 1,//return 语句中不能有赋值表达式
"no-script-url": 0,//禁止使用javascript:void(0)
"no-self-compare": 2,//不能比较自身
"no-sequences": 0,//禁止使用逗号运算符
"no-shadow": 2,//外部作用域中的变量不能与它所包含的作用域中的变量或参数同名
"no-shadow-restricted-names": 2,//严格模式中规定的限制标识符不能作为声明时的变量名使用
"no-spaced-func": 2,//函数调用时 函数名与()之间不能有空格
"no-sparse-arrays": 2,//禁止稀疏数组， [1,,2]
"no-sync": 0,//nodejs 禁止同步方法
"no-ternary": 0,//禁止使用三目运算符
"no-trailing-spaces": 1,//一行结束后面不要有空格
"no-this-before-super": 0,//在调用super()之前不能使用this或super
"no-throw-literal": 2,//禁止抛出字面量错误 throw "error";
"no-undef": 1,//不能有未定义的变量
"no-undef-init": 2,//变量初始化时不能直接给它赋值为undefined
"no-undefined": 2,//不能使用undefined
"no-unexpected-multiline": 2,//避免多行表达式
"no-underscore-dangle": 1,//标识符不能以_开头或结尾
"no-unneeded-ternary": 2,//禁止不必要的嵌套 var isYes = answer === 1 ? true : false;
"no-unreachable": 2,//不能有无法执行的代码
"no-unused-expressions": 2,//禁止无用的表达式
"no-unused-vars": [2, {"vars": "all", "args": "after-used"}],//不能有声明后未被使用的变量或参数
"no-use-before-define": 2,//未定义前不能使用
"no-useless-call": 2,//禁止不必要的call和apply
"no-void": 2,//禁用void操作符
"no-var": 0,//禁用var，用let和const代替
"no-warning-comments": [1, { "terms": ["todo", "fixme", "xxx"], "location": "start" }],//不能有警告备注
"no-with": 2,//禁用with

"array-bracket-spacing": [2, "never"],//是否允许非空数组里面有多余的空格
"arrow-parens": 0,//箭头函数用小括号括起来
"arrow-spacing": 0,//=>的前/后括号
"accessor-pairs": 0,//在对象中使用getter/setter
"block-scoped-var": 0,//块语句中使用var
"brace-style": [1, "1tbs"],//大括号风格
"callback-return": 1,//避免多次调用回调什么的
"camelcase": 2,//强制驼峰法命名
"comma-dangle": [2, "never"],//对象字面量项尾不能有逗号
"comma-spacing": 0,//逗号前后的空格
"comma-style": [2, "last"],//逗号风格，换行时在行首还是行尾
"complexity": [0, 11],//循环复杂度
"computed-property-spacing": [0, "never"],//是否允许计算后的键名什么的
"consistent-return": 0,//return 后面是否允许省略
"consistent-this": [2, "that"],//this别名
"constructor-super": 0,//非派生类不能调用super，派生类必须调用super
"curly": [2, "all"],//必须使用 if(){} 中的{}
"default-case": 2,//switch语句最后必须有default
"dot-location": 0,//对象访问符的位置，换行的时候在行首还是行尾
"dot-notation": [0, { "allowKeywords": true }],//避免不必要的方括号
"eol-last": 0,//文件以单一的换行符结束
"eqeqeq": 2,//必须使用全等
"func-names": 0,//函数表达式必须有名字
"func-style": [0, "declaration"],//函数风格，规定只能使用函数声明/函数表达式
"generator-star-spacing": 0,//生成器函数*的前后空格
"guard-for-in": 0,//for in循环要用if语句过滤
"handle-callback-err": 0,//nodejs 处理错误
"id-length": 0,//变量名长度
"indent": [2, 4],//缩进风格
"init-declarations": 0,//声明时必须赋初值
"key-spacing": [0, { "beforeColon": false, "afterColon": true }],//对象字面量中冒号的前后空格
"lines-around-comment": 0,//行前/行后备注
"max-depth": [0, 4],//嵌套块深度
"max-len": [0, 80, 4],//字符串最大长度
"max-nested-callbacks": [0, 2],//回调嵌套深度
"max-params": [0, 3],//函数最多只能有3个参数
"max-statements": [0, 10],//函数内最多有几个声明
"new-cap": 2,//函数名首行大写必须使用new方式调用，首行小写必须用不带new方式调用
"new-parens": 2,//new时必须加小括号
"newline-after-var": 2,//变量声明后是否需要空一行
"object-curly-spacing": [0, "never"],//大括号内是否允许不必要的空格
"object-shorthand": 0,//强制对象字面量缩写语法
"one-var": 1,//连续声明
"operator-assignment": [0, "always"],//赋值运算符 += -=什么的
"operator-linebreak": [2, "after"],//换行时运算符在行尾还是行首
"padded-blocks": 0,//块语句内行首行尾是否要空行
"prefer-const": 0,//首选const
"prefer-spread": 0,//首选展开运算
"prefer-reflect": 0,//首选Reflect的方法
"quotes": [1, "single"],//引号类型 `` "" ''
"quote-props":[2, "always"],//对象字面量中的属性名是否强制双引号
"radix": 2,//parseInt必须指定第二个参数
"id-match": 0,//命名检测
"require-yield": 0,//生成器函数必须有yield
"semi": [2, "always"],//语句强制分号结尾
"semi-spacing": [0, {"before": false, "after": true}],//分号前后空格
"sort-vars": 0,//变量声明时排序
"space-after-keywords": [0, "always"],//关键字后面是否要空一格
"space-before-blocks": [0, "always"],//不以新行开始的块{前面要不要有空格
"space-before-function-paren": [0, "always"],//函数定义时括号前面要不要有空格
"space-in-parens": [0, "never"],//小括号里面要不要有空格
"space-infix-ops": 0,//中缀操作符周围要不要有空格
"space-return-throw-case": 2,//return throw case后面要不要加空格
"space-unary-ops": [0, { "words": true, "nonwords": false }],//一元运算符的前/后要不要加空格
"spaced-comment": 0,//注释风格要不要有空格什么的
"strict": 2,//使用严格模式
"use-isnan": 2,//禁止比较时使用NaN，只能用isNaN()
"valid-jsdoc": 0,//jsdoc规则
"valid-typeof": 2,//必须使用合法的typeof的值
"vars-on-top": 2,//var必须放在作用域顶部
"wrap-iife": [2, "inside"],//立即执行函数表达式的小括号风格
"wrap-regex": 0,//正则表达式字面量用小括号包起来
"yoda": [2, "never"]//禁止尤达条件

```

### 在 ``package.json``的 scripts配置中增加命令

```php

"lint:es": "eslint -c .eslintrc src/**/*.js --fix",

```

## 2、安装Prettier 到 react 项目

### 需要安装的包如下：

```php

"prettier": "^1.13.5",

```

### 在项目根目录创建配置文件 ``.prettierc``

写入内容：

```php

{
  "printWidth": 100,
  "tabWidth": 2,
  "parser": "babylon",
  "trailingComma": "none",
  "jsxBracketSameLine": true,
  "semi": true,
  "singleQuote": true,
  "overrides": [
    {
      "files": [
        "*.json",
        ".eslintrc",
        ".tslintrc",
        ".prettierrc",
        ".tern-project"
      ],
      "options": {
        "parser": "json",
        "tabWidth": 2
      }
    },
    {
      "files": "*.{css,sass,scss,less}",
      "options": {
        "parser": "postcss",
        "tabWidth": 4
      }
    },
    {
      "files": "*.ts",
      "options": {
        "parser": "typescript"
      }
    }
  ]
}

```

主要配置了：缩进长度是2、单行长度是100、使用单引号等等。


### 在 ``package.json``的 scripts配置中增加命令

```php

"format": "prettier --write"

```

## 3、安装 StyleLint 到 React 项目

### 需要安装的包

```php

"stylelint": "^9.3.0",
"stylelint-config-standard": "^18.2.0",

```

### 在项目根目录创建配置文件 ``.stylelintrc``

写入内容：

```php

{
  "extends": "stylelint-config-standard",
  "rules": {
    "at-rule-empty-line-before": null,
    "at-rule-name-space-after": null,
    "comment-empty-line-before": null,
    "declaration-bang-space-before": null,
    "declaration-empty-line-before": null,
    "declaration-colon-newline-after": null,
    "function-comma-newline-after": null,
    "function-name-case": null,
    "function-parentheses-newline-inside": null,
    "function-max-empty-lines": null,
    "function-whitespace-after": null,
    "value-list-comma-newline-after": null,
    "indentation": null,
    "number-leading-zero": null,
    "number-no-trailing-zeros": null,
    "rule-empty-line-before": null,
    "selector-combinator-space-after": null,
    "selector-list-comma-newline-after": null,
    "selector-pseudo-element-colon-notation": null,
    "unit-no-unknown": null,
    "value-list-max-empty-lines": null,
    "no-empty-source": null,
    "selector-combinator-space-before": null,
    "selector-pseudo-class-no-unknown": null,
    "no-descending-specificity": null,
    "font-family-no-missing-generic-family-keyword": null
  }
}

```


### 在 ``package.json``的 scripts配置中增加命令

```php

  "lint:style": "stylelint \"src/**/*.less\" --syntax less",

```

该项目所有样式都是使用 LESS编写。

# 五、Pre-commit Hook 约束代码提交

上文探讨了 ESLint、Prettier和 StyleLint 在项目中的使用方法和配置文件的编写，这都是针对个人的推荐操作； 为了保证该项目所有的参与者都能统一代码风格， 则需要采用强制约束；假如团队使用 ``Git``昨晚代码托管工具， 在 ``commit``行为和之前进行代码约束， 以便代码质量和风格一致。因此可借助 Husky 和 lint-staged 来实现。

- Husky ：可以方便的让你通过npm scripts来调用各种git hooks。
- lint-staged ：利用git的staged特性，可以提取出本次提交的变动文件，让prettier只处理这些文件。


### 1、配置 npm 脚本命令

在项目的package.json中，配置pre-commit的hook任务：

```php
{
  "scripts": {
    "precommit": "lint-staged",
    "lint-staged:es": "eslint --fix -c .eslintrc",
    "lint-staged:style": "stylelint --syntax less",
  },
  "lint-staged": {
    "src/**/*.{js,jsx}": [
      "format",
      "lint-stagted:es",
      "git add"
    ],
    "src/**/*.less": [
      "format",
      "lint-staged:style",
      "git add"
    ]
  },
}

```

至此，你的项目就可以支持自动检查和格式化了。即使团队中有人没有安装三种代码分析工具，也可以确保他的代码在提交到项目仓库中时，总是被检查修改和格式化了之后。


# 六、在编辑器配置ESLint、Prettier 和 StyleLint

上一部分说到在项目中集成了ESLint、Prettier 和 StyleLint三种代码分析工具， 但每一次还是要手动运行命令。如何能与编辑器（IDE)结合起来呢？以 WebStrom 为例：

## 1、在 WebStorm开启 ESLint

在 WebStorm 中，打开设置（File>Setting或者Alt+F7），按路径进入 ESLint 的配置界面（Languages&Frameworks>JavaScript>Code Quality Tools>ESLint）。开启 ESLint，并配置相应路径，配置文件默认使用.eslintrc。



## 2、在 WebStorm开启 Prettier

在macOS 上快捷键：Alt-Shift-Cmd-P； 在Windows and Linux上快捷键：Alt-Shift-Ctrl-P，可以用 Prettier格式化选中代码、整个文件或目录。


## 3、在 WebStorm开启 StyleLint

WebStorm天然支持stylelint。打开配置() Languages & FromeWorks > StyleSheets > Stylelint)只需在里面开启并配置安装包path即可。

# 六、总结

在使用上述三种代码分析工具后， 代码风格保持一致， 不用再为非业务逻辑代码而争吵， 提高工作效率。由于篇幅问题， 很多东西没有展开描述，可自行查询相关资料。






