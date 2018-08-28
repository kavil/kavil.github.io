---
layout:     post
title:      "NgDragSelectTable"
subtitle:   ""
date:       2018-08-28 14:39:17
author:     "Kai"
header-img: "img/post-bg-re-vs-ng2.jpg"
catalog: true
tags:
    - ng6
    - drag 
    - select
    - table
---

这两天有空 整理了下

[https://github.com/kavil/NgDragSelectTable](https://github.com/kavil/NgDragSelectTable)

# NgDragSelectTable
[![NPM](https://nodei.co/npm/drag-select-table.png?downloads=true&downloadRank=true&stars=true)](https://nodei.co/npm/drag-select-table)

[![NPM Version][npm-image]][npm-url]
[![NPM Downloads][downloads-image]][downloads-url]
[![Node Version][node-image]][node-url]


[npm-image]: https://img.shields.io/npm/v/drag-select-table.svg?style=flat-square
[npm-url]: https://npmjs.org/package/drag-select-table

[downloads-image]: https://img.shields.io/npm/dm/drag-select-table.svg?style=flat-square
[downloads-url]: https://npmjs.org/package/drag-select-table

[node-image]: https://img.shields.io/node/v/phantom.svg?style=flat-square
[node-url]: https://nodejs.org/en/download/


angular 6 component

![预览图](/img/in-post/dragtable.gif)

*This project is a Demo, you can clone it to build.*<br>
*Public component in `projects` folder*<br>

[![Edit ng-drag-select-table](https://codesandbox.io/static/img/play-codesandbox.svg)](https://codesandbox.io/s/github/kavil/NgDragSelectTable/tree/master/)

## Install

```bash
npm install drag-select-table --save
or
yarn add drag-select-table
```

## Usage

```js
import { DragSelectTableModule } from 'drag-select-table';

@NgModule({
  imports: [
    ...
    DragSelectTableModule
  ],
})
```

### simple
It's 5 row and 5 column as default.
```html
<ds-table></ds-table>
```

### emit valuesChange

```js
import { Component } from '@angular/core';

@Component({
    selector: 'drag-select-table-demo',
    template: `<ds-table [row]="row" [column]="col" (valuesChange)="updateValues($event)"></ds-table>`
})
export class MultiSelectTableDemo {
    public row = 7;
    public col = 7;

    public updateValues(values) {
        console.log(values);
    }
}
```

### bidirectional bindings

```js
import { Component } from '@angular/core';

@Component({
    selector: 'drag-select-table-demo',
    template: `<ds-table [row]="row" [column]="col" [(values)]="resTable"></ds-table>`
})
export class MultiSelectTableDemo {
    public row = 7;
    public col = 7;
    public resTable = [];
}
```

## Demo Development server

Run `ng serve` for a dev server. Navigate to `http://localhost:4200/`. The app will automatically reload if you change any of the source files.


## MIT License

Copyright (c) 2018 kai
