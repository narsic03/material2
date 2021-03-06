The `mat-table` provides a Material Design styled data-table that can be used to display rows of 
data. 

This table builds on the foundation of the CDK data-table and uses a similar interface for its 
data input and template, except that its element and attribute selectors will be prefixed 
with `mat-` instead of `cdk-`. For more information on the interface and a detailed look at how
the table is implemented, see the 
[guide covering the CDK data-table](https://material.angular.io/guide/cdk-table).

### Getting Started

<!-- example(table-basic) -->

#### 1. Write your mat-table and provide data

Begin by creating a `<mat-table>` component in your template and passing in data.
  
The simplest way to provide data to the table is by passing a data array to the table's `data` 
input. The table will take the array and render a row for each object in the data array.

```html
<mat-table [dataSource]=”myDataArray”>
  ...
</mat-table>
```

Since the table optimizes for performance, it will not automatically check for changes to the data
array. Instead, when objects are added, removed, or moved on the data array, you can trigger an 
update to the table's rendered rows by calling its `renderRows()` method. 

While an array is the _simplest_ way to bind data into the data source, it is also
the most limited. For more complex applications, using a `DataSource` instance
is recommended. See the section "Advanced data sources" below for more information.

#### 2. Define the column templates

Next, write your table's column templates. 

Each column definition should be given a unique name and contain the content for its header and row 
cells.

Here's a simple column definition with the name `'userName'`. The header cell contains the text 
"Name" and each row cell will render the `name` property of each row's data.

```html
<ng-container matColumnDef="userName">
  <mat-header-cell *matHeaderCellDef> Name </mat-header-cell>
  <mat-cell *matCellDef="let user"> {{user.name}} </mat-cell>
</ng-container>
```

#### 3. Define the row templates

Finally, once you have defined your columns, you need to tell the table which columns will be
rendered in the header and data rows. 

To start, create a variable in your component that contains the list of the columns you want to
render. 

```ts
columnsToDisplay = ['userName', 'age'];
```

Then add `mat-header-row` and `mat-row` to the content of your `mat-table` and provide your
column list as inputs.

```html
<mat-header-row *matHeaderRowDef="columnsToDisplay"></mat-header-row>
<mat-row *matRowDef="let myRowData; columns: columnsToDisplay"></mat-row>
```

Note that this list of columns provided to the rows can be in any order, not necessary the order in
which you wrote the column definitions. Also, you do not necessarily have to include every column
that was defined in your template.

This means that by changing your column list provided to the rows, you can easily re-order and
include/exclude columns dynamically.

### Advanced data sources

The simplest way to provide data to your table is by passing a data array. More complex use-cases 
may benefit from a more flexible approach involving an Observable stream or by encapsulating your 
data source logic into a `DataSource` class.

#### Observable stream of data arrays

An alternative approach to providing data to the table is by passing an Observable stream that emits
the data array to be rendered each time it is changed. The table will listen to this stream and 
automatically trigger an update to the rows each time a new data array is emitted.

#### DataSource

For most real-world applications, providing the table a DataSource instance will be the best way to 
manage data. The DataSource is meant to serve a place to encapsulate any sorting, filtering, 
pagination, and data retrieval logic specific to the application.

A DataSource is simply a base class that has two functions: `connect` and `disconnect`. The 
`connect` function will be called by the table to receive a stream that emits the data array that 
should be rendered. The table will call `disconnect` when the table is destroyed, which may be the 
right time to clean up any subscriptions that may have been registered during the connect process.

### Features

The `MatTable` is focused on a single responsibility: efficiently render rows of data in a 
performant and accessible way.

You'll notice that the table itself doesn't come out of the box with a lot of features, but expects
that the table will be included in a composition of components that fills out its features.

For example, you can add sorting and pagination to the table by using MatSort and MatPaginator and
mutating the data provided to the table according to their outputs.

To simplify the use case of having a table that can sort, paginate, and filter an array of data,
the Angular Material library comes with a `MatTableDataSource` that has already implemented
the logic of determining what rows should be rendered according to the current table state. To add
these feature to the table, check out their respective sections below.

#### Pagination

To paginate the table's data, add a `<mat-paginator>` after the `<mat-table>`. 

If you are using the `MatTableDataSource` for your table's data source, simply provide the 
`MatPaginator` to your data source. It will automatically listen for page changes made by the user 
and send the right paged data to the table.

Otherwise if you are implementing the logic to paginate your data, you will want to listen to the
paginator's `(page)` output and pass the right slice of data to your table.

For more information on using and configuring the `<mat-paginator>`, check out the 
[mat-paginator docs](https://material.angular.io/components/paginator/overview).

The `MatPaginator` is one provided solution to paginating your table's data, but it is not the only 
option. In fact, the table can work with any custom pagination UI or strategy since the `MatTable` 
and its interface is not tied to any one specific implementation.

<!-- example(table-pagination) -->

#### Sorting

To add sorting behavior to the table, add the `matSort` directive to the `<mat-table>` and add 
`mat-sort-header` to each column header cell that should trigger sorting. 

```html
<!-- Name Column -->
<ng-container matColumnDef="position">
  <mat-header-cell *matHeaderCellDef mat-sort-header> Name </mat-header-cell>
  <mat-cell *matCellDef="let element"> {{element.position}} </mat-cell>
</ng-container>
```

If you are using the `MatTableDataSource` for your table's data source, provide the `MatSort` 
directive to the data source and it will automatically listen for sorting changes and change the 
order of data rendered by the table.

By default, the `MatTableDataSource` sorts with the assumption that the sorted column's name 
matches the data property name that the column displays. For example, the following column 
definition is named `position`, which matches the name of the property displayed in the row cell.

Note that if the data properties do not match the column names, or if a more complex data property 
accessor is required, then a custom `sortingDataAccessor` function can be set to override the 
default data accessor on the `MatTableDataSource`.

If you are not using the `MatTableDataSource`, but instead implementing custom logic to sort your 
data, listen to the sort's `(matSortChange)` event and re-order your data according to the sort state. 
If you are providing a data array directly to the table, don't forget to call `renderRows()` on the 
table, since it will not automatically check the array for changes.

<!-- example(table-sorting) -->

For more information on using and configuring the sorting behavior, check out the 
[matSort docs](https://material.angular.io/components/sort/overview).

The `MatSort` is one provided solution to sorting your table's data, but it is not the only option. 
In fact, the table can work with any custom pagination UI or strategy since the `MatTable` and 
its interface is not tied to any one specific implementation.

#### Filtering

Angular Material does not provide a specific component to be used for filtering the `MatTable` 
since there is no single common approach to adding a filter UI to table data.

A general strategy is to add an input where users can type in a filter string and listen to this 
input to change what data is offered from the data source to the table. 

If you are using the `MatTableDataSource`, simply provide the filter string to the  
`MatTableDataSource`. The data source will reduce each row data to a serialized form and will filter 
out the row if it does not contain the filter string. By default, the row data reducing function 
will concatenate all the object values and convert them to lowercase.

For example, the data object `{id: 123, name: 'Mr. Smith', favoriteColor: 'blue'}` will be reduced 
to `123mr. smithblue`. If your filter string was `blue` then it would be considered a match because 
it is contained in the reduced string, and the row would be displayed in the table.

To override the default filtering behavior, a custom `filterPredicate` function can be set which 
takes a data object and filter string and returns true if the data object is considered a match.

<!--- example(table-filtering) -->

#### Selection

Right now there is no formal support for adding a selection UI to the table, but Angular Material 
does offer the right components and pieces to set this up. The following steps are one solution but 
it is not the only way to incorporate row selection in your table.

##### 1. Add a selection model

Get started by setting up a `SelectionModel` from `@angular/cdk/collections` that will maintain the 
selection state.

```js
const initialSelection = [];
const allowMultiSelect = true;
this.selection = new SelectionModel<MyDataType>(allowMultiSelect, initialSelection);
```

##### 2. Define a selection column

Add a column definition for displaying the row checkboxes, including a master toggle checkbox for 
the header. The column name should be added to the list of displayed columns provided to the 
`<mat-header-row>` and `<mat-row>`.

```html
<ng-container matColumnDef="select">
  <mat-header-cell *matHeaderCellDef>
    <mat-checkbox (change)="$event ? masterToggle() : null"
                  [checked]="selection.hasValue() && isAllSelected()"
                  [indeterminate]="selection.hasValue() && !isAllSelected()">
    </mat-checkbox>
  </mat-header-cell>
  <mat-cell *matCellDef="let row">
    <mat-checkbox (click)="$event.stopPropagation()"
                  (change)="$event ? selection.toggle(row) : null"
                  [checked]="selection.isSelected(row)">
    </mat-checkbox>
  </mat-cell>
</ng-container>
```

##### 3. Add event handling logic

Implement the behavior in your component's logic to handle the header's master toggle and checking 
if all rows are selected.

```js
/** Whether the number of selected elements matches the total number of rows. */
isAllSelected() {
  const numSelected = this.selection.selected.length;
  const numRows = this.dataSource.data.length;
  return numSelected == numRows;
}

/** Selects all rows if they are not all selected; otherwise clear selection. */
masterToggle() {
  this.isAllSelected() ?
      this.selection.clear() :
      this.dataSource.data.forEach(row => this.selection.select(row));
}
```

##### 4. Include overflow styling 

Finally, adjust the styling for the select column so that its overflow is not hidden. This allows 
the ripple effect to extend beyond the cell.

```css
.mat-column-select {
  overflow: initial;
}
```

<!--- example(table-selection) -->

### Accessibility
Tables without text or labels should be given a meaningful label via `aria-label` or 
`aria-labelledby`. The `aria-readonly` defaults to `true` if it's not set.

Table's default role is `grid`, and it can be changed to `treegrid` through `role` attribute.

`mat-table` does not manage any focus/keyboard interaction on its own. Users can add desired 
focus/keyboard interactions in their application.
