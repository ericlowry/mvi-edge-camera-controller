# Alert Templates

Alerts are Go Templates that can be simple text or include various metadata available in an inspection pipeline. Templates can be very simple or very complex. The [official documentation](https://golang.org/pkg/text/template/) is highly technical. A less technical introduction to templates can be found [here](https://blog.gopheracademy.com/advent-2017/using-go-templates/)

Everything in a template, including line feeds, is processed as simple text unless enclosed in double curly braces - `{{}}`

Alert templates will reference metadata using a `{{.type.key}}` notation.

## Metadata Types

There are three types of metadata that are available to be referenced in templates (examples for referencing them are provided in the next section) :

- Pipeline context variables (`.context.Metadata`)
  - Pipeline contexts always include the following variables:
    - InspectionUUID
    - InspectionName
    - StationName
    - DeviceUUID
    - DeviceName
    - MVIVersion
    - ModelUUID

- Inspection Rule metadata (`.result`, `.image`)
  - The internal metadata produced when a rule is triggered by inference metadata. The following example shows the structure of rule metadata:

    ```
    result:
    {
        "Name": "failed two",  <=== from the rule configuration
        "UUID": "7995469f-62fc-4cbf-9ba8-f1ba1c3feac0",
        "Result": "fail",  <=== from the rule configuration
        "ObjectLabel": "two",  <=== from the rule configuration
        "FilteredResults": [  <=== an array of objects that match the rule's ObjectLabel
            {
                "Rectangle":
                {
                    "Min":
                    {
                        "X": 532,
                        "Y": 162
                    },
                    "Max":
                    {
                        "X": 600,
                        "Y": 297
                    }
                },
                "Score": 0.9381611347198486,
                "Label":
                {
                    "Name": "two"
                }
            }
        ],
        "ObjectCount": 1  <=== number of FilteredResults
        ]
    },

    image:
    {   
        ID: "15713525972.jpg"
    }
    ```

  - In addition to the rule results, the image filename of the image that is being processed is available as `.image.ID`. The location of this image is `<install root>/vision-edge/volume/run/images/<InspectionUUID>/<image filename>`. 

    - A full path to the image can be included in an alert message as:



       `<install root>/vision-edge/volume/run/images/{{.context.Metadata.InspectionUUID}}/{{.image.ID}}`.

    - A URL to the image can be included in an alert message as:



       `https://<controller hostname>/opt/ibm/vision-edge/images/{{.context.Metadata.InspectionUUID}}/{{.image.ID}}`.

  

- Trigger metadata (`.trigger`)

  - In pipelines with an MQTT trigger, the MQTT messages are available to be refernced in templates.

    - MQTT trigger messages are arbitrary so it is not possible to provide an enumeration of referencable values here.

    - The messages must be in JSON format in order to be referenced in templates





## Examples



### Simple Text



Any text in a template will be output by the template exactly as defined, include line breaks. Eg if the alert template is defined as 

``` 

{

    "Inspection" : "Test 1",

    "Result" : "fail"

} 

 ```

 That will be the output alert message.



 ### Context Metadata



To use the configured name of the Inspection in the example above instead of a text literal, we use the `{{.type.key}}` notation and the template would be defined as:

``` 

{

    "Inspection" : "{{.context.Metadata.InspectionName}}",

    "Result" : "fail"

}

```

Assuming the Inspection's name is `Test 1`, the output would be identical to the simple text example. 



### Result Metadata



To continue with the example above, to use the rule result instead of the text literal, the template would be defined as:

```

{

    "Inspection" : "{{.context.Metadata.InspectionName}}",

    "Result" : "{{.result.Result}}"

}

```

The output would be the same as in the previous two examples.



A more complex example would be a template of:

```

{

 "station":"{{.context.Metadata.Station}}",

 "inspection":"{{.context.Metadata.Inspection}}",

 "rule":"{{.result.Name}}",

 "result":"{{.result.Result}}",

 "imageUrl":"https://vision.edge.org/opt/ibm/vision-edge/images/{{.context.Metadata.InspectionUUID}}/{{.image.ID}}",

 "objectcount":"{{.result.ObjectCount}}",

 "objects":[

    {{range $j, $obj := .result.FilteredResults}}{{if gt $j 0}},

	{{end}}{

     "label":"{{$obj.Label.Name}}",

     "score":{{$obj.Score}}

    }{{end}}

  ]

}

```

Which would produce the output:

```

{

 "station":"Test",

 "inspection":"Test 1",

 "rule":"failed two",

 "result":"fail",

 "imageUrl":"https://vision.edge.org/opt/ibm/vision-edge/images/78d6d7ab-2e67-4d36-a7e3-66a2b73f86c6/15713525972.jpg",

 "objectcount":"1",

 "objects":[

    {

     "label":"two",

     "score":0.9381611347198486

    }

  ]

}

```

This complex example uses a number of Go Template features:

- `station`, `inspection`, `rule`, `result`, and `objectcount` all use the notation described above, where a value is referenced from either `context.Metadata`, `result`, or `image` types using the `{{.type.key}}` notation:

    ```

    {

      "station":"{{.context.Metadata.Station}}",

      "inspection":"{{.context.Metadata.Inspection}}",

      "rule":"{{.result.Name}}",

      "result":"{{.result.Result}}",

      "imageUrl":"https://vision.edge.org/opt/ibm/vision-edge/images/{{.context.Metadata.InspectionUUID}}/{{.image.ID}}",

      "objectcount":"{{.result.ObjectCount}}",

    ```

- Since FilteredResults is an array, the `range` action is used to iterate over the array:

    ```

    {{range $j, $obj := .result.FilteredResults}}

    ```

    `$j` and `$obj` are template `variables` where `$j` is the index into the array and `$obj` is the array element. Note that we still use `.type.key` to reference the FilteredResults array.

- Next we use a conditional to insert a comma and line break between json elements:

    ```

    {{if gt $j 0}},

	{{end}}

    ```

    Since `$j` is the index into the array, the comma and line feed are not output before the first element in the array but are output before each subsequent element. We don't see the effect of this in our output as there is only one element in the array, but this is the most common way to separate elements in a JSON array when using Go Templates.

- Since `$obj` is an element in the FilteredResults array, we reference the label and score as 

    ``` 

    {

     "label":"{{$obj.Label.Name}}",

     "score":{{$obj.Score}}

    }

    ```

    Note that in the result metadata `Name` is nested within `Label`. To reference nested elements, we simply use additional dots. 

- Likewise, if we wanted to include the rectangle coordinates, we would reference them as 

  - `{{$obj.Rectangle.Min.X}}`

  - `{{$obj.Rectangle.Min.Y}}`

  - `{{$obj.Rectangle.Max.X}}`

  - `{{$obj.Rectangle.Max.Y}}`

- Finally, we close our range, objects array, and JSON object:

    ```

        {{end}}

      ]

    }

    ```

 Note that the `{{end}}` terminates the `range` action whereas the previous `{{end}}` terminated the `{{if gt $j 0}}` conditional.



 ### Trigger Metadata

 Trigger metadata is referenced using the `{{.type.key}}` notation.



 For example, if an MQTT trigger message such as:

   ```

   {

       "object" : "widget",

       "serial_id" : "152637"

   }

   ```

is received by the pipeline, we could reference it in an alert template along with context and result metadata as follows:

   ```

   {

       "inspection":"{{.context.Metadata.Inspection}}",

       "rule":"{{.result.Name}}",

       "result":"{{.result.Result}}",

       "object_type" : "{{.trigger.object}},"

       "serial_num" : "{{.trigger.serial_id}}"

   }

   ```

and the alert output would be:

   ```

   {

       "inspection":"Test 1",

       "rule":"failed two",

       "result":"fail",

       "object_type" : "widget",

       "serial_num" : "152637"

   }

   ```