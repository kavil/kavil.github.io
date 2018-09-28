---
layout:     post
title:      "Angular之FormArray全探索"
subtitle:   ""
date:       2018-09-18 14:13:05
author:     "Kai"
header-img: "img/contact-bg.jpg"
catalog: true
tags:
    - ng6
    - FormArray 
---


## 前提

大家都知道Angular中表单创建有两种方式<br>
一种是 `Template-Driven` 模板驱动<br>
另一种是 `Reactive/Model-Driven` 动态表单

还一个基础<br>
`FormControl`<br>
`FormArray`<br>
`FormGroup`<br>
这三者都是继承自`AbstractControl`这个基本class<br>
本是同一类东西 可偏偏这个`formArray`不支持模板驱动的写法<br>
官方17年2月就加入了TODO list 到现在还没解决你敢信<br>
无奈只能用动态表单。<br>
需要注意的是FormArray的结构需要FormGroup作为父级。下面代码中可以看到写法<br>

## 如何直接把FormArray放进模板驱动的form里
上面说过 formArray 没有模板驱动方式<br>
但我们可以把其封装成单独的组件，然后用父级form的api加入进去

```typescript
import { Component, OnInit, Input } from "@angular/core";
import { FormBuilder, NgForm, FormArray, FormGroup } from "@angular/forms";

@Component({
  selector: "array-form",
  template: `
      <ng-container [formGroup]="formGroup">
        <button type="button" (click)="addItem()">add</button>
        <button type="button" (click)="removeItem()">remove</button>
        <div formArrayName="formArray" *ngFor="let item of formArray.controls; let i = index;">
            <ng-container [formGroupName]="i"><br>
                <input formControlName="name" type="text" placeholder="name"><br>
                <input formControlName="phone" type="text" placeholder="phone">
            </ng-container>
        </div>
      </ng-container><br><br><br>
    `
})
export class ArrayFormComponent implements OnInit {
  @Input() name: string;
  @Input() form: NgForm; // 把父级的form拿到 用于插入这个父级form
  formGroup: FormGroup; // formArray需要formGroup这个中间件才能循环
  formArray: FormArray;

  constructor(private formBuilder: FormBuilder) {}

  ngOnInit() {
    let array = [];
    this.formArray = this.formBuilder.array(array);
    this.formGroup = this.formBuilder.group({
      formArray: this.formArray
    });
    this.form.control.addControl(this.name, this.formArray);
    // 直接插入array层 不需要中间那层FormGroup 但你又不能抛弃formGroup 因为渲染模板需要formGroup
  }
  initItem(): FormGroup {
    const fg = this.formBuilder.group({
      name: "",
      phone: ""
    });
    return fg;
  }

  addItem() {
    const formArray = <FormArray>this.formGroup.controls["formArray"];
    formArray.push(this.initItem());
  }

  removeItem(index: number) {
    const formArray = <FormArray>this.formGroup.controls["formArray"];
    formArray.removeAt(index);
  }
}

```

接下来是父组件 就可以直接按模板驱动的写法

```typescript
import { Component } from "@angular/core";
import { NgForm } from "@angular/forms";

@Component({
  selector: "app-root",
  template: `
    <form #detailForm="ngForm" (ngSubmit)="submit(detailForm)">
        form表单：<br>
        <input name="id" ngModel type="text" placeholder="id"><br>
        <array-form name="detail" [form]="detailForm"></array-form>
        <button type="submit">submit</button>
    </form>
  `
})
export class AppComponent {
  submit(form: NgForm) {
    const params = form.value; // 模板驱动取值就是这么简洁
    //TODO submit to api
    console.log(params);
  }
}
```

![预览图](/img/in-post/formarray.png)
<br>全部代码如下<br>
[![Edit Angular之FormArray](https://codesandbox.io/static/img/play-codesandbox.svg)](https://codesandbox.io/s/l2pvzxkmz7)