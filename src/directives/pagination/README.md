# Pagination Directive

## Another one?

Yes, there are quite a few pagination solutions for Angular out there already, but what I wanted to do was make
something that would be truly plug-n-play - no need to do any set-up or logic in your controller. Just add
an attribute, drop in your navigation wherever you like, and boom - instant, full-featured pagination.

## Demo

[Here is a working demo on Plunker](http://plnkr.co/edit/Wtkv71LIqUR4OhzhgpqL?p=preview) which demonstrates some cool features such as live-binding the "itemsPerPage" and
filtering of the collection.

## Example

Let's say you have a collection of items on your controller's `$scope`. Often you want to display them with
the `ng-repeat` directive and then paginate the results if there are too many to fit on one page. This is what this
module will enable:

```HTML
<ul>
    <li dir-paginate="item in items | itemsPerPage: 10">{{ item }}</li>
</ul>

// then somewhere else on the page ....

<dir-pagination-controls></dir-pagination-controls>
```

...and that's literally it.

## Usage

First you need to include the module in your project:

```JavaScript
// in your app
angular.module('myApp', ['angularUtils.directives.dirPagination']);
```

Then create the paginated content:
```HTML
<ANY
    dir-paginate="expression | itemsPerPage: (int|expression) [: paginationId (string literal)]"
    [current-page=""]
    [pagination-id=""]
    [total-items=""]>
    ...
    </ANY>
```
And finally include the pagination itself.


```HTML
...
<dir-pagination-controls
    [max-size=""]
    [direction-links=""]
    [boundary-links=""]
    [on-page-change=""]
    [pagination-id=""]
    [template-url=""]>
    </dir-pagination-controls>
```

### `dir-paginate`

* **`expression`** Under the hood, this directive delegates to the `ng-repeat` directive, so the syntax for the
expression is exactly as you would expect. [See the ng-repeat docs for the full rundown](https://docs.angularjs.org/api/ng/directive/ngRepeat).
This means that you can also use any kind of filters you like, etc.

* **`itemsPerPage`** The `expression` **must** include this filter. It is required by the pagination logic. The syntax
is the same as any filter: `itemsPerPage: 10`, or you can also bind it to a property of the $scope: `itemsPerPage: pageSize`. **Note:** This filter should come *after* any other filters in order to work as expected. A safe rule is to always put it at the end of the expression.
The optional third argument `paginationId` is used when you need more than one independent pagination instance on one page. See the section below
on setting up multiple instances.

* **`current-page`** (optional) Specify a property on your controller's $scope that will be bound to the current
page of the pagination. If this is not specified, the directive will automatically create a property named `__currentPage` and use
that instead.

* **`pagination-id`** (optional) Used to group together the dir-paginate directive with a corresponding dir-pagination-controls when you need more than
one pagination instance per page. See the section below on setting up multiple instances.

* **`total-items`** When working with asynchronous data (i.e. data that is paginated on the server and sent one page at a time to the client), you would be sent
only one page of results and then some meta-data containing the total number of results. In this case, the pagination directive would think that your one page
of result was the full dataset, and therefore no pagination is needed. To prevent the default behaviour, you need to specify the `total-items` attribute, which
will then be used to calculate the pagination. For more information see the section on working with asynchronous data below.

### `dir-pagination-controls`

* **`max-size`** (optional, default = 9) Specify a maximum number of pagination links to display. The default is 9, and
the minimum is 5 (setting a lower value than 5 will not have an effect).

* **`direction-links`** (optional, default = true) Specify whether to display the "forwards" & "backwards" arrows in the
pagination.

* **`boundary-links`** (optional, default = false) Specify whether to display the "start" & "end" arrows in the
pagination.

* **`on-page-change`** (optional, default = null) Specify a callback method to run each time one of the pagination links is clicked. The method will be passed the
argument `newPageNumber`, which is an integer equal to the page number that has just been navigated to. **Note** you must use that exact argument name in your view,
i.e. `<dir-pagination-controls on-page-change="myMethod(newPageNumber)">`, and the method you specify must be defined on your controller $scope.

* **`pagination-id`** (optional) Used to group together the dir-pagination-controls with a corresponding dir-paginate when you need more than
one pagination instance per page. See the section below on setting up multiple instances.

* **`template-url`**  (optional, default = `directives/pagination/dirPagination.tpl.html`) Specifies the template to use.

Note: you cannot use the `dir-pagination-controls` directive without `dir-paginate`. Attempting to do so will result in an
exception.


## Multiple Pagination Instances on One Page

Multiple instances of the directives may be included on a single page by specifying a `pagination-id`. This property **must** be specified in **3** places
for this to work:

1. Specify the `pagination-id` attribute on the `dir-paginate` directive.
2. Specify the third parameter of the `itemsPerPage` filter.
3. Specify the `pagination-id` attribute on the `dir-paginations-controls` directive.

An example of two independent paginations on one page would look like this:

```HTML
<!-- first pagination instance -->
<ul>
    <li dir-paginate="customer in customers | itemsPerPage: 10: 'cust'" pagination-id="cust">{{ customer.name }}</li>
</ul>

<dir-pagination-controls pagination-id="cust"></dir-pagination-controls>

<!-- second pagination instance -->
<ul>
    <li dir-paginate="branch in branches | itemsPerPage: 10: 'branch'" pagination-id="branch">{{ customer.name }}</li>
</ul>

<dir-pagination-controls pagination-id="branch"></dir-pagination-controls>
```

The pagination-ids above are set to "cust" in the first instance and "branch" in the second. The pagination-ids can be anything you like,
the important thing is to make sure the exact same id is used in all 3 places. If the 3 ids don't match, you should see a helpful
exception in the console.

### Demo

Here is a working demo featuring two instances on one page: [http://plnkr.co/edit/Pm4L53UYAieF808v8wxL?p=preview](http://plnkr.co/edit/Pm4L53UYAieF808v8wxL?p=preview)


## Working With Asynchronous Data

The arrangement described above works well for smaller collections, but once your data set reaches a certain size, you may not want to have to get the entire collection
from the server just to view a few pages. The solution to this is to paginate on the server-side, whereby the server will only send one page of data at a time. In this case,
the directive would see the small (one page) data set and think that was all, resulting in no pagination at all (assuming you set the `itemsPerPage` filter to match the number
of items returned per server-side page).

The solution is to use the `total-items` attribute on the `dir-paginate` directive. Commonly, in such a server-side paging scenario, the server would also return some meta-data
along with the result set, specifying the total number of results. A typical example would look like this:

```JavaScript
{
    Count: 1400,
    Items: [
        { // item 1... },
        { // item 2... },
        { // item 3... },
        ...
        { // item 25... }
    ]
}
```

In this case, the server is returning the first *page* - 25 results - of a set that totals 1400 results. Therefore we must tell the directive that, even though it can only see
25 items in the collection right now, there are actually a total of 1400 items, so it should therefore make 1400/25 = **56** pagination links.

Of course, once the user clicks on page 2, you will need to make a new request to the server to fetch the results for page 2. This will have to be implemented by you in your
controller, but you will want to either 1) use the `on-page-change` callback to trigger the request or 2) set up a $watch on the property you specified in the `current-page`
attribute. Either of those methods will allow you to call a function whenever the page changes, which will fetch the new page of results. The second method has the
potential advantage of being triggered whenever the current-page changes, rather than only when the pagination links are clicked.

### Example Asynchronous Setup

```JavaScript
.controller('UsersController', function($scope, $http) {
    $scope.users = [];
    $scope.totalUsers = 0;
    $scope.usersPerPage = 25; // this should match however many results your API puts on one page
    getResultsPage(1);

    $scope.pagination = {
        current: 1
    };

    $scope.pageChanged = function(newPage) {
        getResultsPage(newPage);
    };

    function getResultsPage(pageNumber) {
        // this is just an example, in reality this stuff should be in a service
        $http.get('path/to/api/users?page=' + pageNumber)
            .then(function(result) {
                $scope.users = result.data.Items;
                $scope.totalUsers = result.data.Count
            });
    }
})
```

```HTML
<div ng-controller="UsersController">
    <table>
        <tr dir-paginate="user in users | itemsPerPage: 25" total-items="totalUsers">
            <td>{{ user.name }}</td>
            <td>{{ user.email }}</td>
        </tr>
    </table>

    <dir-pagination-controls on-page-change="pageChanged(newPageNumber)"></dir-pagination-controls>
</div>
````

## Styling

I've based the pagination navigation on the Bootstrap 3 component, so if you use Bootstrap in your project,
you'll get some nice styling for free. If you don't use Bootstrap, it's simple to style the links with css.

## Credits

I did quite a bit of research before I figured I needed to make my own directive, and I picked up a lot of good ideas
from various sources:

* Daniel Tabuenca: https://groups.google.com/d/msg/angular/an9QpzqIYiM/r8v-3W1X5vcJ. This is where I learned how to
delegate to the ng-repeat directive from within my own, and I used some of the code he gives here.

* AngularUI Bootstrap: https://github.com/angular-ui/bootstrap. I used a few ideas, and a couple of attribute names,
from their pagination directive.

* StackOverflow: http://stackoverflow.com/questions/10816073/how-to-do-paging-in-angularjs. Picked up a lot of ideas
from the various contributors to this thread.
