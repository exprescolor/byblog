---
layout:     post
title:      DevExpress使用记录
subtitle:   DevExpress使用记录
date:       2024-12-25
author:     BY
header-img: img/post-bg-desk.jpg
catalog: true
tags:
    - Winform
    - DevExpress
---


## 下载

[DevExpress下载地址](URL_ADDRESS[DevExpress下载地址](https://www.devexpress.com/downloads/winforms/)

## 安装
* 下载DevExpress安装包后根据需求勾选安装，我是默认全部安装。

## 创建自定义的范围输入控件 (UserControl)
* 首先，我们需要创建一个自定义的 UserControl，用于在下拉框中输入范围。这个控件将包含两个 SpinEdit 控件，分别用于输入最小值和最大值。

## 使用DevExpress做自定义的数据展示控件，并支持多选操作。

  PopupContainerEdit：
  * PopupContainerEdit 是 DevExpress 提供的一个控件，可以用来显示一个弹出窗口，其中可以包含其他控件。
  * 我们可以使用 PopupContainerEdit 来创建一个自定义的控件，用于展示一个树形结构。
  * 在 PopupContainerEdit 中，我们可以使用 TreeList 控件来展示树形结构。
  * 我们可以使用 TreeList 的 Nodes 属性来添加树形结构的节点。
  * 我们可以使用 TreeList 的 SelectedNodes 属性来获取选中的节点。
  * 我们可以使用 TreeList 的 CheckedNodes 属性来获取选中的节点。
  
    作用：这是一个编辑控件，用户可以点击它来显示一个弹出窗口。弹出窗口的内容由 PopupControlContainer 控件承载。
    放置位置：通常放置在窗体上的任意位置，例如 (30, 30)。

  PopupControlContainer：
  * 作用：这是一个容器控件，用于承载弹出窗口的内容。
  * 放置位置：放置在窗体上，与 PopupContainerEdit 同一窗体中，但它本身不会直接显示在界面上，而是作为弹出内容存在。
  TreeList：
  * 作用：这是一个控件，用于展示树形结构。
  * 放置位置：放置在窗体上，与 PopupContainerEdit 同一窗体中，但它本身不会直接显示在界面上，而是作为弹出内容存在。
  Button：
  * 作用：这是一个按钮控件，用户可以点击它来执行一些操作。
  * 放置位置：放置在窗体上，与 PopupContainerEdit 同一窗体中，但它本身不会直接显示在界面上，而是作为弹出内容存在。

  作用：这是一个容器控件，用于承载弹出窗口的内容，如 TreeList 和 Button。
  放置位置：放置在窗体上，与 PopupContainerEdit 同一窗体中，但它本身不会直接显示在界面上，而是作为弹出内容存在。

# 步骤详解

## 拖拽以下控件到 Form1：

* PopupContainerEdit：用于显示已选择的项。
* PopupControlContainer：作为弹出容器，承载 TreeList 控件。
* TreeList：用于展示父子级别的数据，并支持多选。
* Button：用于确认选择并关闭弹出窗口。

## 设置控件属性：

* PopupContainerEdit (popupContainerEdit1)：

  * Name: popupContainerEdit1
  * Location: (30, 30)
  * Size: (300, 24)
  * Properties.TextEditStyle: DisableTextEditor
  * Properties.PopupControl: popupControlContainer1

* PopupControlContainer (popupControlContainer1)：

  * Name: popupControlContainer1
  * Location: (30, 60)
  * Size: (300, 400)
  * BorderStyle: Simple
  * Visible: False（默认隐藏）
  * Controls: 将 TreeList 和 Button 添加到此容器内。

* TreeList (treeList1)：

  * Name: treeList1
  * Dock: Fill
  * OptionsView.ShowCheckBoxes: True
  * OptionsBehavior.AllowRecursiveNodeChecking: True
  * OptionsSelection.MultiSelect: True
  * ParentFieldName: ParentID
  * KeyFieldName: ID
  * Columns: 添加一列用于显示节点名称。
    * FieldName: Name
    * Caption: 名称
    * Visible: True

* Button (buttonConfirm)：

  * Name: buttonConfirm
  * Text: 确定
  * Dock: Bottom
  * Height: 30

**注意：** 确保 PopupContainerEdit 和 PopupControlContainer 是独立的控件，都直接添加到窗体（Form）的控件集合中，而不是相互嵌套。