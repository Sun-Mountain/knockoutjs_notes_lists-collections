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
```