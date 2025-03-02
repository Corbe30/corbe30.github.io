---
date: '2025-02-27T22:09:04+05:30'
draft: false
title: 'Optimizing Graphs for Spreadsheet Formulas Evaluation'
tags: ["excel", "fortune-sheet", "algorithm"]
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: ""
canonicalURL: "https://canonical.url/to/page"
disableHLJS: true # to disable highlightjs
disableShare: true
disableHLJS: false
hideSummary: false
searchHidden: false
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: false
ShowWordCount: false
ShowRssButtonInSectionTermList: true
UseHugoToc: true
---

Cells in a spreadsheet can form a chain of interdependent formulas, where a change in one cell triggers the reevaluation of the entire chain. For example, in the example below, changing cell `A1` or `B1` recomputes `C1`, which recomputes `C2` and `D1`.

|       | **A** | **B** | **C**   | **D**   |
| ----- | ----- | ----- | ------- | ------- |
| **1** | 4     | 5     | =A1+B1  | =B1+C1  |
| **2** |       |       | =C1 + 1 | =C2 + 1 |

I'll explain how Fortunesheet handles this by using **Topological Sorting**, and how I [optimized its performance by 82%](https://github.com/ruilisi/fortune-sheet/pull/649).

## An Overview of Steps

The following steps are involved in handling chained formula revaluation:

```javascript
function revaluateFormulas(modifiedCells, sheets) {
  // 1. gather dependencies
  const allFormulas = getAllFormulas(sheets);
  const formulasInfoMap = getFormulasInfo(allFormulas);

  // 2. Find root cells for re-evaluation
  const rootCells = filterFormulaDependencyCells(
    modifiedCells,
    formulasInfoMap
  );

  // 3. Find all affected formulas
  const affectedFormulas = topologicalSort(rootCells, formulasInfoMap);
  evalAllFormuals(affectedFormulas);
}
```

## 1. Gathering Dependencies

```javascript
const allFormulas = getAllFormulas(sheets);
const formulasInfoMap = getFormulasInfo(allFormulas);
```

`getAllFormulas()` takes all the sheets are arguments. This is because formulas can refer to cells from another sheet as well. This data is cached. It's only updated whenever there's a write/update/delete operation of a formula cell.

`getFormulasInfo()` returns an object containing:

```json
{
  "=C1+1": {
    "key": "r2-c2-iabcd-efgh",
    "children": ["r1-c3-iabcd-efgh"], // ["C2"]
    "cellDependency": [
      { // C1
        "row": [0, 0],
        "column": [2, 2],
        "sheetId": "abcd-efgh"
      }
    ],
  },
  ...
}
```
* `key`: unique id of the cell, created by combining row, column and sheetId
* `cellDependency`: all the cell ranges on which our formula depends
* `children`: formulas which depend on this formula

Previously, `cellDependecy` was calculated for every formula update, which would cause edit operations to take ~5sec on sheets with 1,00,000+ formulas. I optimized it by caching `formulasInfoMap` at the cost of ~30mb for such sheets, which brought down the edit time to 0.8sec.

`children` is calculated by looping through `cellDependency`. If any of the cell in `cellDependency` is a formula, we update that cell's `children` array to include current formula:
```javascript
for (formula in allFormulas) {
    for (cell in formula.cellDependency) {
        if (isFormula(cell)) cell.children.push(formula);
    }
}
```

The `key` and `cellDependency` properties is cached, but `children` isn't. Every `children` is recalculated on every formula update. This is because, if one of the child cell is deleted, we would need to handle updation of cached reference too. Of course, this could be solved by maintaining a `parents` array, but it [hasn't been implemented](https://github.com/ruilisi/fortune-sheet/issues/686) yet.


## 2. Finding Root Cells

```javascript
const rootCells = filterFormulaDependencyCells(
    modifiedCells,
    formulasInfoMap
);
```

`rootCells` are simply the `modifiedCells` on which any formula is dependent. In other words, we filter the cells whose updation leads to no other change in the sheet. 
`filterFormulaDependencyCells()` works by looping through `cellDependency` in `formulasInfoMap` of each formula, and checking if it contains any of the `modifiedCells`. 

Even this can be cached by maintaining a reverse map of cell vs formula, but `filterFormulaDependencyCells()` isn't performance intensive, and the cost of memory would be high.

## 3. Topological Sorting

```javascript
const affectedFormulas = topologicalSort(rootCells, formulasInfoMap);
```

Let's take another example:
|       | **A** | **B** | **C**   | **D**   | **E**   |
| ----- | ----- | ----- | ------- | ------- | ------- |
| **1** | 4     | 5     | =A1+B1  | =B1+C1  | =C1+D1  |


<img src="https://picsum.photos/200/300" />

We can see why we need to use topological sorting instead of DFS. Calculating `E1` before `D1` will give incorrect ouput.
There are several ways to implement topological sort, but keep in mind that our formulas could be cyclic, so we cannot assume a DAG.

---

That's it! It's a common misconception that frontend programming doesn't involve data structures and algorithms, but it really depends on the product we are building.