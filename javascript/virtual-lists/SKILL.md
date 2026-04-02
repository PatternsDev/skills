---
name: virtual-lists
description: Optimize rendering of large lists by only displaying items visible in the viewport (windowing).
license: MIT
metadata:
  author: patterns.dev
  version: "1.1"
paths:
  - "**/*.js"
  - "**/*.ts"
related_skills:
  - "module-pattern"
  - "singleton-pattern"
---

# List Virtualization

List virtualization (also known as windowing) is the idea of rendering only visible rows of content in a dynamic list instead of the entire list. The rows rendered are only a small subset of the full list with what is visible (the window) moving as the user scrolls. This can improve rendering performance.

If you use React and need to **display large lists of data efficiently**, you may be familiar with [react-virtualized](https://bvaughn.github.io/react-virtualized/). It's a windowing library by [Brian Vaughn](https://twitter.com/brian_d_vaughn) that renders only the items currently visible in a list (within a scrolling "viewport"). This means you don't need to pay the cost of thousands of rows of data being rendered at once.

## When to Use

- Use this when rendering large lists or grids (hundreds/thousands of items) that cause performance issues
- This is helpful for reducing initial render time and improving scroll performance

## Instructions

- Use `react-window` (or `react-virtualized`) to render only visible items in a scrollable container
- Choose `FixedSizeList` for items of equal height or `VariableSizeList` for items of different heights
- Use `react-window-infinite-loader` for incrementally loading data as the user scrolls
- Consider CSS `content-visibility: auto` for simpler cases where full virtualization isn't needed

## Details

### How does list virtualization work?

"Virtualizing" a list of items involves **maintaining a window** and **moving that window around your list**. Windowing in react-virtualized works by:

- Having a small container DOM element (e.g `<ul>`) with relative positioning (window)
- Having a big DOM element for scrolling
- Absolutely positioning children inside the container, setting their styles for top, left, width and height.

Rather than rendering 1000s of elements from a list at once (which can cause slower initial rendering or impact scroll performance), **virtualization focuses on rendering just items visible to the user**.

This can help keep list rendering fast on mid to low-end devices. You can fetch/display more items as the user scrolls, unloading previous entries and replacing them with new ones.

### A smaller alternative to react-virtualized

[react-window](https://react-window.now.sh/) is a rewrite of react-virtualized by the same author aiming to be **smaller**, faster and more [tree-shakeable](https://developers.google.com/web/fundamentals/performance/optimizing-javascript/tree-shaking/).

In a tree-shakeable library, size is a function of which API surfaces you choose to use. You can see ~20-30KB (gzipped) savings using it in place of react-virtualized.

The APIs for both packages are similar and where they differ, react-window tends to be simpler. react-window's components include:

#### List

Lists render a **windowed list (row) of elements** meaning that only the visible rows are displayed to users (e.g [FixedSizeList](https://react-window.now.sh/#/examples/list/fixed-size), [VariableSizeList](https://react-window.now.sh/#/examples/list/variable-size)). Lists use a Grid (internally) to render rows, relaying props to that inner Grid.

**Rendering a list of data using React**

Here's an example of rendering a list of simple data (`itemsArray`) using React:

```js
import React from "react";
import ReactDOM from "react-dom";

const itemsArray = [
  { name: "Drake" },
  { name: "Halsey" },
  { name: "Camillo Cabello" },
  { name: "Travis Scott" },
  { name: "Bazzi" },
  { name: "Flume" },
  { name: "Nicki Minaj" },
  { name: "Kodak Black" },
  { name: "Tyga" },
  { name: "Buno Mars" },
  { name: "Lil Wayne" }, ...
]; // our data

const Row = ({ index, style }) => (
  <div className={index % 2 ? "ListItemOdd" : "ListItemEven"} style={style}>
    {itemsArray[index].name}
  </div>
);

const Example = () => (
  <div
    style={{
      height: 150,
      width: 300
    }}
    class="List"
  >
    {itemsArray.map((item, index) => Row({ index }))}
  </div>
);

ReactDOM.render(<Example />, document.getElementById("root"));
```

**Rendering a list using react-window**

...and here's the same example using react-window's `FixedSizeList`, which takes a few props (`width`, `height`, `itemCount`, `itemSize`) and a row rendering function passed as a child:

```js
import React from "react";
import ReactDOM from "react-dom";
import { FixedSizeList as List } from "react-window";

const itemsArray = [...]; // our data

const Row = ({ index, style }) => (
  <div className={index % 2 ? "ListItemOdd" : "ListItemEven"} style={style}>
    {itemsArray[index].name}
  </div>
);

const Example = () => (
  <List
    className="List"
    height={150}
    itemCount={itemsArray.length}
    itemSize={35}
    width={300}
  >
    {Row}
  </List>
);

ReactDOM.render(<Example />, document.getElementById("root"));
```

You can try out `FixedSizeList` on [CodeSandbox](https://codesandbox.io/s/github/bvaughn/react-window/tree/master/website/sandboxes/fixed-size-list-vertical).

#### Grid

Grid renders **tabular data** with virtualization along the vertical and horizontal axes (e.g [FixedSizeGrid](https://react-window.now.sh/#/examples/grid/fixed-size), [VariableSizeGrid](https://react-window.now.sh/#/examples/grid/variable-size)). It only renders the Grid cells needed to fill itself based on current horizontal/vertical scroll positions.

```js
import React from 'react';
import ReactDOM from 'react-dom';
import { FixedSizeGrid as Grid } from 'react-window';

const itemsArray = [
  [{},{},{},...],
  [{},{},{},...],
  [{},{},{},...],
  [{},{},{},...],
];

const Cell = ({ columnIndex, rowIndex, style }) => (
  <div
    className={
      columnIndex % 2
        ? rowIndex % 2 === 0
          ? 'GridItemOdd'
          : 'GridItemEven'
        : rowIndex % 2
          ? 'GridItemOdd'
          : 'GridItemEven'
    }
    style={style}
  >
    {itemsArray[rowIndex][columnIndex].name}
  </div>
);

const Example = () => (
  <Grid
    className="Grid"
    columnCount={5}
    columnWidth={100}
    height={150}
    rowCount={5}
    rowHeight={35}
    width={300}
  >
    {Cell}
  </Grid>
);

ReactDOM.render(<Example />, document.getElementById('root'));
```

You can also try out `FixedSizeGrid` on [CodeSandbox](https://codesandbox.io/s/github/bvaughn/react-window/tree/master/website/sandboxes/fixed-size-grid).

### More in-depth react-window examples

[Scott Taylor](https://github.com/staylor) implemented an open-source [Pitchfork music reviews scraper](http://pitchfork.highforthis.com/) [(src)](https://github.com/staylor/pitchfork-scraper) using `react-window` and `FixedSizeGrid`.

Pitchfork scraper uses [react-window-infinite-loader](https://github.com/bvaughn/react-window-infinite-loader) ([demo](https://codesandbox.io/s/5wqo7z2np4)) which helps break large data sets down into chunks that can be loaded as they are scrolled into view.

Here's a snippet of how react-window-infinite-loader is incorporated in this app:

```js
import React, { Component } from 'react';
import { FixedSizeGrid as Grid } from 'react-window';
import InfiniteLoader from 'react-window-infinite-loader';

// ...

render() {
  return (
    <InfiniteLoader
      isItemLoaded={this.isItemLoaded}
      loadMoreItems={this.loadMoreItems}
      itemCount={this.state.count + 1}
    >
      {({ onItemsRendered, ref }) => (
        <Grid
          onItemsRendered={this.onItemsRendered(onItemsRendered)}
          columnCount={COLUMN_SIZE}
          columnWidth={180}
          height={800}
          rowCount={Math.max(this.state.count / COLUMN_SIZE)}
          rowHeight={220}
          width={1024}
          ref={ref}
        >
          {this.renderCell}
        </Grid>
      )}
    </InfiniteLoader>
  );
}
```

You might find the [commit](https://github.com/staylor/pitchfork-scraper/commit/d9bff69e332ad9de8351c67f4848fc7968209eff) porting the app over from `react-virtualized` useful.

An implementation using `FixedSizeList` is also available:

```js
return (
  <InfiniteLoader
    isItemLoaded={this.isItemLoaded}
    loadMoreItems={this.loadMoreItems}
    itemCount={this.state.count}
  >
    {({ onItemsRendered, ref }) => (
      <section>
        <FixedSizeList
          itemCount={this.state.count}
          itemSize={ROW_HEIGHT}
          onItemsRendered={onItemsRendered}
          height={this.state.height}
          width={this.state.width}
          ref={ref}
        >
          {this.renderCell}
        </FixedSizeList>
      </section>
    )}
  </InfiniteLoader>
);
```

For even more complex needs, a [The Movie Database](https://www.themoviedb.org/) [demo app](https://tmdb-viewer.surge.sh/) used react-virtualized and Infinite Loader under the hood. [Porting](https://github.com/addyosmani/tmdb-viewer/blob/master/src/components/InfiniteMoviesList.js) it over to react-window and react-window-infinite-loader didn't take long, but we did discover a few components were not yet supported. Regardless, the final functionality is [pretty close](https://tmdb-viewer.firebaseapp.com/).

The missing components were WindowScroller and AutoSizer, which we'll look at next.

```js
// ...
return (
  <section>
    <AutoSizer disableHeight>
      {({width}) => {
        const {movies, hasMore} = this.props;
        const rowCount = getRowsAmount(width, movies.length, hasMore);
        // ...
        return (
          <InfiniteLoader
            ref={this.infiniteLoaderRef}
            // ...
            {({onRowsRendered, registerChild}) => (
              <WindowScroller>
                {({height, scrollTop}) => (
```

### What's missing from react-window?

react-window does not yet have the complete API surface of react-virtualized, so do check the [comparison docs](https://github.com/bvaughn/react-window#how-is-react-window-different-from-react-virtualized) if considering it. What's missing?

- [WindowScroller](https://github.com/bvaughn/react-virtualized/blob/master/docs/WindowScroller.md) - This is a `react-virtualized` component that enables Lists to be scrolled based on the window's scroll positions. There are currently [no plans](https://github.com/bvaughn/react-window/issues/30) to implement this for react-window so you'll need to solve this in userland.
- [AutoSizer](https://github.com/bvaughn/react-virtualized/blob/master/docs/AutoSizer.md) - HOC that grows to fit all of the available space, automatically adjusting the width and height of a single child. Brian implemented this as a [standalone](https://www.npmjs.com/package/react-virtualized-auto-sizer) package. Follow [this issue](https://github.com/bvaughn/react-window/issues/5) for the latest.
- [CellMeasurer](https://github.com/bvaughn/react-virtualized/blob/master/docs/CellMeasurer.md) - HOC automatically measuring a cell's content by rendering it in a way that is not visible to the user. Follow [here](https://github.com/bvaughn/react-window/issues/6) for discussion on support.

That said, we found react-window sufficient for most of our needs with what it includes out of the box.

### Improvements in the web platform

Some modern browsers now support [CSS content-visibility](https://web.dev/content-visibility/). `content-visibility:auto` allows you to skip rendering & painting offscreen content until needed. If you have a long HTML document with costly rendering, consider trying the property out.

For rendering lists of dynamic content, I still recommend using a library like react-window. It would be hard to have a `content-visibility:hidden` version of such a library that beats a version aggressively using `display:none` or removing DOM nodes when offscreen like many list virtualization libraries may do today.

### Further reading

For further reading about react-window and react-virtualized, check out:

- [Rendering performant lists with react-window](https://alligator.io/react/lists-with-react-window/)
- [Creating More Efficient React Views with Windowing](https://www.youtube.com/watch?v=t4tuhg7b50I)
- [Rendering lists with react-virtualized](https://css-tricks.com/rendering-lists-using-react-virtualized/)
- [Rendering large lists with react-virtualized](https://blog.logrocket.com/rendering-large-lists-with-react-virtualized-82741907a6b3)

## Source

- [patterns.dev/vanilla/virtual-lists](https://patterns.dev/vanilla/virtual-lists)
