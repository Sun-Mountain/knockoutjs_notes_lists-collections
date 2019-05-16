# KnockoutJS Notes: Lists & Collections

Create repeating blocks of UI elements.

In this example we'll create a table for seat and meal reservations.

#### HTML
```html
<h2>Your seat reservations</h2>

<table>
    <thead><tr>
        <th>Passenger name</th><th>Meal</th><th>Surcharge</th><th></th>
    </tr></thead>
    <!-- Todo: Generate table body -->
    
    // Using FOREACH binding, render a copy of its child elements for each entry in the seats array

    <tbody data-bind="foreach: seats">
        <tr>
            <td data-bind="text: name"></td>
            <td data-bind="text: meal().mealName"></td>
            <td data-bind="text: meal().price"></td>
        </tr>
    </tbody>
</table>
```

#### JavaScript
```JavaScript
// Class to represent a row in the seat reservations grid
function SeatReservation(name, initialMeal) {
    var self = this;
    self.name = name;
    self.meal = ko.observable(initialMeal);
}

// Overall viewmodel for this screen, along with initial state
function ReservationsViewModel() {
    var self = this;

    // Non-editable catalog data - would come from the server
    self.availableMeals = [
        { mealName: "Standard (sandwich)", price: 0 },
        { mealName: "Premium (lobster)", price: 34.95 },
        { mealName: "Ultimate (whole zebra)", price: 290 }
    ];    

    // Editable data
    self.seats = ko.observableArray([
        new SeatReservation("Steve", self.availableMeals[0]),
        new SeatReservation("Bert", self.availableMeals[0])
    ]);
}

ko.applyBindings(new ReservationsViewModel());
```

**Note**: `foreach` is part of a family of control flow bindings that includes `foreach`, `if`, `ifnot`, and `with`. 


Let's add a button after the `</table>` tag in the HTML document to reserve seats & meals.

#### HTML
```html
    <button data-bind="click: addSeat">Reserve</button>
```

Now we need to add an `addSeat` function so the button *does* something.

#### JavaScript
```JavaScript
function ReservationsViewModel() {
    // ... leave all the rest unchanged ...

    // Operations
    self.addSeat = function() {
        self.seats.push(new SeatReservation("", self.availableMeals[0]));
    }
}
```

But now we need to be able to change existing and additions to the list.

#### HTML
```HTML
<tbody data-bind="foreach: seats">
    <tr>
        <td><input data-bind="value: name" /></td>
        <td><select data-bind="options: $root.availableMeals, value: meal, optionsText: 'mealName'"></select></td>
        <td data-bind="text: meal().price"></td>
    </tr>    
</tbody>
```

This code uses two new bindings, `options` and `optionsText`, which together control both the set of available items in a dropdown list, and which object property (in this case, `mealName`) is used to represent each item on screen.

We've got some great OOP going, but now let's add some custom formatting for the price by using `ko.computed` (so it will update automatically whenever the meal selection changes).

```JavaScript
function SeatReservation(name, initialMeal) {
    var self = this;
    self.name = name;
    self.meal = ko.observable(initialMeal);

    self.formattedPrice = ko.computed(function() {
        var price = self.meal().price;
        return price ? "$" + price.toFixed(2) : "None";        
    });
}
```

Then update the view for the data-bind in the HTML document for price:

```HTML
<tr>
    <td><input data-bind="value: name" /></td>
    <td><select data-bind="options: $root.availableMeals, value: meal, optionsText: 'mealName'"></select></td>
    <td data-bind="text: formattedPrice"></td>
</tr>
```

#### Removing Items & Showing Final Surcharge
It's only logical once you have the ability to add to a list, to be able to remove.

We'll do this by adding a link to each generated item in the table row.

```html
<tr>
    <td><input data-bind="value: name" /></td>
    <td><select data-bind="options: $root.availableMeals, value: meal, optionsText: 'mealName'"></select></td>
    <td data-bind="text: formattedPrice"></td>
    <td><a href="#" data-bind="click: $root.removeSeat">Remove</a></td>
</tr>         
```

The `$root.` prefix makes Knockout look for a `removeSeat` handler on your top-level viewmodel instead of the `SeatReservation` instance, which is more convenient.

```JavaScript
function ReservationsViewModel() {
    // ... leave the rest unchanged ...

    // Operations
    self.addSeat = function() { /* ... leave unchanged ... */ }
    self.removeSeat = function(seat) { self.seats.remove(seat) }
}    
```


WOOOOOO! Almost done with these notes!

#### Displaying total surcharge

Let's add another `ko.computed` property in the `ReservationsViewModel` AFTER the `seats` `observables`.

```JavaScript
self.totalSurcharge = ko.computed(function() {
   var total = 0;
   for (var i = 0; i < self.seats().length; i++)
       total += self.seats()[i].meal().price;
   return total;
});
```

Notice that `seats` and `meal` are both observables.

```HTML
<h3 data-bind="visible: totalSurcharge() > 0">
    Total surcharge: $<span data-bind="text: totalSurcharge().toFixed(2)"></span>
</h3>
```


#### Final Thoughts

If you want to add a limit to how many times you can add a reservation all you have to do is add an enable binding:

```JavaScript
<button data-bind="click: addSeat, enable: seats().length < 5">Reserve another seat</button>
```

The button becomes disabled when the seat limit is reached. You don't have to write any code to re-enable it when the user removes some seats (cluttering up your "remove" logic), because the expression will automatically be re-evaluated by Knockout when the associated data changes.