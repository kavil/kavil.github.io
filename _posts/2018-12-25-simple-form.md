---
date:       2018-09-18 14:13:05
author:     "Kai"
---

> 这篇文章假设你已经会使用angular的ngForm。

# 前提
大家都知道Angular中表单创建有两种方式<br>
一种是 `Template-Driven` 模板驱动<br>
一种是 `Reactive/Model-Driven` 动态表单<br>
<br>
还一个基础<br>
`FormControl`<br>
`FormArray`<br>
`FormGroup`<br>
这三者都是继承自`AbstractControl`这个基本class<br>

好了<br>
回想下我们刚开始学前端最原始、最简单的form表单提交方式。<br>
<br>
对了<br>
就是这种清爽的表单<br>
```html
<form>
    <input type="text" name="username" placeholder="name">
    <input type="password" name="pass" placeholder="pass">

    <button type="reset">重置</button>
    <button type="submit">登录</button>
</form>
```
![登录](/img/in-post/login.png)

# 从后端接口要post的数据格式开始

一般的，要达到像上面那种清爽的提交方式，angular中使用`Template-Driven` 模板驱动就可以完全搞定<br>
like this
```html
<form #loginForm="ngForm" (ngSubmit)="submitFunction(loginForm)">
    <input type="text" [(ngModel)]="username" name="username" placeholder="name">
    <input type="password" [(ngModel)]="pass" name="pass" placeholder="pass">

    <button type="reset">重置</button>
    <button type="submit">登录</button>
</form>
```
然后可以闭着眼睛把form里的value提交给接口了
```js
submitFunction(formModel: NgForm){
    params = {...formModel.value};
}
```

事实上，对于复杂的表单需求，后端接口肯定不会整齐的要个扁平化的json<br>
它可能是这样--里面有多级对象<br>
like this `localtion`
```js
params = {
    province: '广东',
    localtion: {
        lat: 111.111,
        lng: 222.222,
    }
}
```

它也可能是这样--里面有数组（这是最骚的）<br>
like this `city`
```js
params = {
    province: '广东',
    localtion: {
        lat: 111.111,
        lng: 222.222,
    },
    city: [
        {name: '广州', id: 1},
        {name: '深圳', id: 2}
    ]
}
```
对于这种需求，首先想到的肯定是多写几个表单，然后到submitFunction里写一堆转换逻辑<br>
需要提交的不多还能看得懂，一旦多了过几天自己都看不懂就难受了(╯﹏╰)。<br>
要想达到那种清爽的、闭着眼睛提交表单，我们就要考虑封装组件、使用`Reactive/Model-Driven` 动态表单封装。<br>

# 多级对象

先来解决里面有多级对象的<br>
## 方法一
使用NG_VALUE_ACCESSOR将组件封装成类似input的表单组件，这样使用该组件就像使用input一样<br>

```js
import { Component, forwardRef, Input, OnInit } from '@angular/core';
import { NG_VALUE_ACCESSOR, ControlValueAccessor, AbstractControl } from '@angular/forms';

const INPUT_CONTROL_VALUE_ACCESSOR: any = {
    provide: NG_VALUE_ACCESSOR,
    useExisting: forwardRef(() => LocationComponent),
    multi: true
};

@Component({
    selector: 'localtion',
    providers: [INPUT_CONTROL_VALUE_ACCESSOR],
    template: `
        <input type="text" [(ngModel)]="localtion.lat" (ngModelChange)="valueChange()" name="lat" placeholder="lat"><br>
        <input type="text" [(ngModel)]="localtion.lng" (ngModelChange)="valueChange()" name="lng" placeholder="lng">
    `
})
export class LocationComponent implements OnInit, ControlValueAccessor {

    @Input() value: any = {};

    valueChange() {
        this.onChangeCallback(this.value);
    }

    public writeValue(event: any): void {
        if (event) {
            this.value = event;
        }
    }

    public registerOnChange(fn: any): void {
        this.onChangeCallback = fn;
    }

    public registerOnTouched(fn: any): void {
        this.onTouchedCallback = fn;
    }

    private onTouchedCallback = () => {
        // placeholder
    }

    private onChangeCallback = (_: any) => {
        // placeholder
    }
}
```

然后父组件就可以直接使用了，接着愉快的闭着眼睛提交^_^
```html
<form #myForm="ngForm" (ngSubmit)="submitFunction(myForm)">
    <input type="text" [(ngModel)]="province" name="province" placeholder="province">
    <localtion [(ngModel)]="localtion" name="localtion"></localtion>

    <button type="submit">提交</button>
</form>
```

## 方法二
使用`Reactive/Model-Driven` 动态表单封装

```js
import { Component, OnInit, Input } from "@angular/core";
import { FormBuilder, NgForm, FormArray, FormGroup } from "@angular/forms";

@Component({
  selector: "localtion-reactive",
  template: `
      <ng-container [formGroup]="formGroup">
        <input formControlName="lat" type="text" placeholder="lat"><br>
        <input formControlName="lng" type="text" placeholder="lng">
      </ng-container>
    `
})
export class LocaltionReactiveComponent implements OnInit {
  @Input() name: string;
  @Input() form: NgForm; // 把父级的form传入 用于插入这个父级form
  formGroup: FormGroup; 

  constructor(private formBuilder: FormBuilder) {}

  ngOnInit() {
    this.formGroup = this.formBuilder.group({
      lat: "",
      lng: ""
    });
    this.form.control.addControl(this.name, this.formGroup);
  }
```

然后父组件就可以直接使用了，接着愉快的闭着眼睛提交^_^
```html
<form #myForm="ngForm" (ngSubmit)="submitFunction(myForm)">
    <input type="text" [(ngModel)]="province" name="province" placeholder="province">
    <localtion-reactive [form]="myForm" name="localtion"></localtion-reactive>

    <button type="submit">提交</button>
</form>
```
## 两种方式的选择
第一种方式的原理是把组件封装成一个form元素，而第二种方式是直接插入到父组件的form对象中去。<br>
细心的同学发现了，这个demo写下来明显第一种更佳<br>
对的，第一种适合封装仅多一级的对象，想象下如果里面有更深的多级。。。封死你<br>
第二种虽然在这个demo上显得鸡肋，但其扩展性是最佳的<br>
<br>
# 最骚的数组封装

显然要想封装出个数组，上面方法一的纯`Template-Driven` 模板驱动是行不通的，不是别的原因，纯粹是官方弄不出来/(ㄒoㄒ)/~~<br>
官方17年2月就加入了TODO list 到现在还没解决你敢信<br>
<br>
这里我们只能使用`Reactive/Model-Driven` 动态表单封装了<br>

```js
import { Component, OnInit, Input } from "@angular/core";
import { FormBuilder, NgForm, FormArray, FormGroup } from "@angular/forms";

@Component({
  selector: "city-array",
  template: `
      <ng-container [formGroup]="formGroup">
        <button type="button" (click)="addItem()">add</button>
        <button type="button" (click)="removeItem()">remove</button>
        <div formArrayName="formArray" *ngFor="let item of formArray.controls; let i = index;">
            <ng-container [formGroupName]="i"><br>
                <input formControlName="id" type="text" placeholder="city id"><br>
                <input formControlName="name" type="text" placeholder="city name">
            </ng-container>
        </div>
      </ng-container><br>
    `
})
export class ArrayFormComponent implements OnInit {
  @Input() name: string;
  @Input() form: NgForm; // 把父级的form传入 用于插入这个父级form
  formGroup: FormGroup; // formArray需要formGroup这个中间件才能在模板循环
  formArray: FormArray;

  constructor(private formBuilder: FormBuilder) {}

  ngOnInit() {
    let array = [];
    this.formArray = this.formBuilder.array(array);
    this.formGroup = this.formBuilder.group({
      formArray: this.formArray
    });
    this.form.control.addControl(this.name, this.formArray);
    // 直接插入array层 不需要中间那层FormGroup 但你又不能抛弃formGroup 因为渲染模板需要formGroup 官方规定的 气不气
  }
  initItem(): FormGroup {
    const fg = this.formBuilder.group({
      id: "",
      name: ""
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

需要注意的是 formArray与formGroup的关系，如果上面没跳过 formGroup，那中间会多一层对象包含这个数组。<br>


然后父组件就可以直接使用了，接着愉快的闭着眼睛提交^_^<br>
```html
<form #myForm="ngForm" (ngSubmit)="submitFunction(myForm)">
    <input type="text" [(ngModel)]="province" name="province" placeholder="province"><br />
    <localtion [(ngModel)]="localtion" name="localtion"></localtion>
    <city-array [form]="myForm" name="city"></city-array>
    <br />
    <button type="submit">提交</button>
</form>
```

<br>全部代码如下<br>
[![Edit 最复杂却又最简单的Angular表单](https://codesandbox.io/static/img/play-codesandbox.svg)](https://codesandbox.io/s/y04k6lno8v)

![res](/img/in-post/res.png)


## 祝大家圣诞🎄快乐