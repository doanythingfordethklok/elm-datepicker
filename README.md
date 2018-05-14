# Elm Date Picker

A Material Design Date Picker for the lovely Elm language.

[Check out the demo here](http://abradley2.github.io/elm-datepicker/)

![pretty image](https://i.imgur.com/3o2WoMc.png)

### Usage

Since this is a material date picker, you will need to have material-icons included.
Also include the `DatePicker.css` found in the root of this directory.

```
<link href="https://fonts.googleapis.com/icon?family=Material+Icons" rel="stylesheet">
<link rel="stylesheet" type="text/css" href="DatePicker.css" />
```

You can also load up the Roboto font, but despite this being a material-style date picker,
it is not required.

### TEA

**Message**  
You will first need to add the `DatePickerMsg` to the type consumed by your `update` function so
it recognizes this type.
```
import DatePicker exposing (DatePickerMsg, DatePickerMsg(..))

type Msg
    = FireZeMissiles
    | HandleDatePickerMsg DatePickerMsg
```
Note that the `DateSelected` message gives two dates for it's arguments. The first is the selected date,
the second is the date that was selected previously.

**Init**  

Initialize a date picker and add it to your model. Do not throw out the generated command.

```
import DatePicker exposing (datePickerInit)

init : (Model, Cmd Msg)
init =
    let
        (datePickerData, datePickerInitCmd) =
            datePickerInit "my-datepicker-id"
    in
        ({ datePickerData = datePickerData
         , ...
         }
         , Cmd.map HandleDatePickerMsg datePickerInitCmd
        )
```

**Model**  

The DatePickerModel type needs to be added to any data structure that requires a picker instance
```
import DatePicker exposing (DatePickerModel)

type alias Model =
    { username : String
    , datePickerData : DatePickerModel
    }
}

init : Model
...
```

This is mostly an opaque type, but there are still fields you may care about as the
date picker encapsulates it's own "selected date" field.
```
getSelectedDate datePickerModel =
    datePickerData.selectedDate
```

You can also retrieve this from the `DateSelected` or `SubmitClicked` message

**Update**  

`datePickerUpdate` consumes the message you've mapped and a `DatePickerModel` to output `( DatePickerModel, Cmd DatePickerMsg)`.
You will need to alter your update function to handle `DatePickerMsg`'s that flow through and allow it to update accordingly.
```
import DatePicker exposing (datePickerUpdate, DatePickerMsg(SelectDate))

update : Msg -> Model -> ( Model, Cmd Msg )
update msg model =
    case msg of
        NoOp ->
            ( model, Cmd.none )

        OnDatePickerMsg datePickerMsg ->
            -- first let's allow the date picker to update itself
            datePickerUpdate datePickerMsg model.datePickerData
                |> (\( data, cmd ) ->
                    ( { model | datePickerData = data }
                    , Cmd.map OnDatePickerMsg cmd
                    )
                )
                -- and now we can respond to any internal messages we want
                |> (\( model, cmd ) ->
                    case datePickerMsg of
                        SubmitClicked currentSelectedDate ->
                            ( { model | selectedDate = Just currentSelectedDate }
                            , cmd
                            )

                        _ ->
                            ( model, cmd )
                )
```

**View**  

Now that we're all wired up we can render it in our view!
```
view : Model -> Html Msg
view model =
     div
        []
        [ datePickerView
              model.datePickerData
              { canSelect = (\date -> True)
              }
          |> Html.map OnDatePickerMsg
        ]
```

### Customization

Navigate to the root of this directory, edit some variables in `styl/Variables.styl` and
run `npm install && npm run build-styles`. Then grab the css as you would before.