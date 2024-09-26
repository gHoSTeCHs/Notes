The word `[uppercase](https://angular.io/api/common/UpperCasePipe)` in the interpolation binding after the pipe `|` character, activates the built-in `UppercasePipe`.

[Pipes](https://angular.io/guide/pipes-overview) are a good way to format strings, currency amounts, dates, and other display data. Angular ships with several built-in pipes, and you can create your own.

### Data Binding
`[([ngModel](https://angular.io/api/forms/NgModel))]` is Angular's two-way data binding syntax.

Here it binds the `hero.name` property to the HTML text box so that data can flow _in both directions_. Data can flow from the `hero.name` property to the text box and from the text box back to the `hero.name`.

### Angular repeater

```ts
<li *ngFor="let hero of heroes">
```

The [`*ngFor`](https://angular.io/guide/built-in-directives#ngFor) is Angular's _repeater_ directive. It repeats the host element for each element in a list.

The syntax in this example is as follows:

|SYNTAX|DETAILS|
|:--|:--|
|`<li>`|The host element.|
|`heroes`|Holds the mock heroes list from the `HeroesComponent` class, the mock heroes list.|
|`hero`|Holds the current hero object for each iteration through the list.|

Don't forget to put the asterisk `*` in front of `[ngFor](https://angular.io/api/common/NgFor)`. It's a critical part of the syntax.