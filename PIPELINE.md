# An in-depth guide to the ALeRCE pipeline

Every step works as a "event driven microservice", which listens to a topic and must receive one or more messages in order to perform their operations. The pipeline steps are developed under the [APF Framework](https://github.com/alercebroker/APF). This framework gives some strong guidelines on the structure of the step, in addition of providing some hooks which facilitates the parsing of the information received by the step and the further deployment as well.  

## Pipeline Diagram (detailed)

![Image](https://user-images.githubusercontent.com/20263599/229163793-f0cefe89-6a2b-4dee-a111-20da2eec3461.png)

## Steps specification

### Sorting Hat

The Sorting Hat is a gateway step which assigns an ALeRCE ID (aid) to every incoming alert. This `aid` can be newly generated or an already existing one, depending on the result of the crossmatch performed on the object database. 
At the present time, this step must be deployed once per survey, with different settings to work with the given survey. So you might find more than one instance of sorting hat deployed.

#### Input Schema

```python
{
    "type": "record",
    "doc": "Multi stream alert of any telescope/survey",
    "name": "alerce.alert",
    "fields": [
        {"name": "oid", "type": "string"},
        {"name": "tid", "type": "string"},
        {"name": "pid", "type": "long"},
        {"name": "candid", "type": ["long", "string"]},
        {"name": "mjd", "type": "double"},
        {"name": "fid", "type": "int"},
        {"name": "ra", "type": "double"},
        {"name": "dec", "type": "double"},
        {"name": "mag", "type": "float"},
        {"name": "e_mag", "type": "float"},
        {"name": "isdiffpos", "type": "int"},
        {"name": "e_ra", "type": "float"},
        {"name": "e_dec", "type": "float"},
        {
            "name": "extra_fields",
            "type": EXTRA_FIELDS,
        },
        {"name": "aid", "type": "string"},
        {"name": "stamps", "type": STAMPS},
    ],
}
```

Given the **STAMPS** type:

```python
{
    "type": "record",
    "name": "stamps",
    "fields": [
        {"name": "science", "type": ["null", "bytes"], "default": None},
        {"name": "template", "type": ["null", "bytes"], "default": None},
        {"name": "difference", "type": ["null", "bytes"], "default": None},
    ],
}
```

and the **EXTRA_FIELDS** type:

```python
{
    "type": "map",
    "values": ["null", "int", "float", "string", "bytes"],
    "default": {},
}
```

### Stamp Classifier Step

#### Input Schema

### Previous Candidates Step

Parses the `extra_fields` of the detections to retrieve the binary field `prv_candidates` if the alert comes from ZTF. Otherwise, it does nothing. Then, this step returns a list with the processed objects, including their `aid`, previous candidates, detections without the `extra_fields` field and their non_detections.

#### Input Schema

```python
{
  "type": "record",
  "doc": "Previous candidates schema with new alert and previous detections and non detections",
  "name": "prv_candidates",
  "fields": [
    {
      "name": "aid",
      "type": "string"
    },
    {
      "name": "new_alert",
      "type": {
        "type": "record",
        "name": "alert",
        "fields": [
          {
            "name": "oid",
            "type": "string"
          },
          {
            "name": "tid",
            "type": "string"
          },
          {
            "name": "pid",
            "type": "long"
          },
          {
            "name": "candid",
            "type": ["long", "string"]
          },
          {
            "name": "mjd",
            "type": "double"
          },
          {
            "name": "fid",
            "type": "int"
          },
          {
            "name": "ra",
            "type": "double"
          },
          {
            "name": "dec",
            "type": "double"
          },
          {
            "name": "mag",
            "type": "float"
          },
          {
            "name": "e_mag",
            "type": "float"
          },
          {
            "name": "isdiffpos",
            "type": "int"
          },
          {
            "name": "e_ra",
            "type": "float"
          },
          {
            "name": "e_dec",
            "type": "float"
          },
          {
            "name": "extra_fields",
            "type": {
              "default": {},
              "type": "map",
              "values": ["null", "int", "float", "string", "bytes"]
            }
          },
          {
            "name": "aid",
            "type": "string"
          }
        ]
      }
    },
    {
      "name": "prv_detections",
      "type": {
        "type": "array",
        "items": "alert",
        "default": []
      }
    },
    {
      "name": "non_detections",
      "type": {
        "type": "array",
        "items": {
          "type": "record",
          "name": "non_detection",
          "fields": [
            {
              "name": "aid",
              "type": "string"
            },
            {
              "name": "tid",
              "type": "string"
            },
            {
              "name": "oid",
              "type": "string"
            },
            {
              "name": "mjd",
              "type": "double"
            },
            {
              "name": "fid",
              "type": "int"
            },
            {
              "name": "diffmaglim",
              "type": "int"
            }
          ]
        }
      },
      "default": []
    }
  ]
}
```

### Lightcurve Step 

This step retrieves from the database and then returns the entire lightcurve of an object.
It merges the repeated objects within the same batch of messages.

#### Input Schema

```python
{
  "type": "record",
  "doc": "Lightcurve schema with detections and non-detections",
  "name": "Lightcurve",
  "fields": [
    {
      "name": "aid",
      "type": "string"
    },
    {
      "name": "detections",
      "type": {
        "type": "array",
        "items": {
            "type": "record",
            "name": "alert",
            "fields": [
              {
                  "name": "aid",
                  "type": "string"
              },
              {
                  "name": "oid",
                  "type": "string"
              },
              {
                  "name": "tid",
                  "type": "string"
              },
              {
                  "name": "candid",
                  "type": ["long", "string"]
              },
              {
                  "name": "mjd",
                  "type": "double"
              },
              {
                  "name": "fid",
                  "type": "int"
              },
              {
                  "name": "ra",
                  "type": "double"
              },
              {
                  "name": "dec",
                  "type": "double"
              },
              {
                  "name": "mag",
                  "type": "float"
              },
              {
                  "name": "e_mag",
                  "type": "float"
              },
              {
                  "name": "isdiffpos",
                  "type": "int"
              },
              {
                  "name": "e_ra",
                  "type": "float"
              },
              {
                  "name": "e_dec",
                  "type": "float"
              },
              {
                  "name": "extra_fields",
                  "type": {
                  "default": {},
                  "type": "map",
                  "values": ["null", "int", "float", "string", "bytes"]
                  }
              }
            ]
        },
        "default": []
      }
    },
    {
      "name": "non_detections",
      "type": {
        "type": "array",
        "items": {
          "type": "record",
          "name": "non_detection",
          "fields": [
            {
              "name": "aid",
              "type": "string"
            },
            {
              "name": "tid",
              "type": "string"
            },
            {
              "name": "oid",
              "type": "string"
            },
            {
              "name": "mjd",
              "type": "double"
            },
            {
              "name": "fid",
              "type": "int"
            },
            {
              "name": "diffmaglim",
              "type": "int"
            }
          ]
        }
      },
      "default": []
    }
  ]
}
```

### Correction Step

Process alerts and applies correction to pass from difference magnitude to apparent magnitude. This includes the previous detections coming from the ZTF alerts.

#### Input Schema

```
{
  "type": "record",
  "doc": "Previous candidates schema with new alert and previous detections and non detections",
  "name": "prv_candidates",
  "fields": [
    {
      "name": "aid",
      "type": "string"
    },
    {
      "name": "detections",
      "type": {
        "type": "array",
        "items": {
        "type": "record",
        "name": "alert",
        "fields": [
          {
            "name": "candid",
            "type": ["long", "string"]
          },
          {
            "name": "tid",
            "type": "string"
          },
          {
            "name": "aid",
            "type": "string"
          },
          {
            "name": "oid",
            "type": "string"
          },
          {
            "name": "mjd",
            "type": "double"
          },
          {
            "name": "fid",
            "type": "int"
          },
          {
            "name": "pid",
            "type": "long"
          },
          {
            "name": "ra",
            "type": "double"
          },
          {
            "name": "e_ra",
            "type": "float"
          },
          {
            "name": "dec",
            "type": "double"
          },
          {
            "name": "e_dec",
            "type": "float"
          },
          {
            "name": "mag",
            "type": "float"
          },
          {
            "name": "e_mag",
            "type": "float"
          },
          {
            "name": "mag_corr",
            "type": ["float", "null"]
          },
          {
            "name": "e_mag_corr",
            "type": ["float", "null"]
          },
          {
            "name": "e_mag_corr_ext",
            "type": ["float", "null"]
          },
          {
            "name": "isdiffpos",
            "type": "int"
          },
          {
            "name": "corrected",
            "type": "boolean"
          },
          {
            "name": "dubious",
            "type": "boolean"
          },
          {
            "name": "has_stamp",
            "type": "boolean"
          },
          {
            "name": "stellar",
            "type": "boolean"
          },
          {
            "name": "extra_fields",
            "type": {
              "default": {},
              "type": "map",
              "values": ["null", "int", "float", "string", "bytes", "boolean"]
            }
          }
        ]
      },
        "default": []
      }
    },
    {
      "name": "non_detections",
      "type": {
        "type": "array",
        "items": {
          "type": "record",
          "name": "non_detection",
          "fields": [
            {
              "name": "aid",
              "type": "string"
            },
            {
              "name": "tid",
              "type": "string"
            },
            {
              "name": "oid",
              "type": "string"
            },
            {
              "name": "mjd",
              "type": "double"
            },
            {
              "name": "fid",
              "type": "int"
            },
            {
              "name": "diffmaglim",
              "type": "int"
            }
          ]
        }
      },
      "default": []
    }
  ]
}
```

### Magstats Step

Calculates some important statistics of the object.

#### Input Schema

```python
{
  "type": "record",
  "doc": "Schema for magstats step, with detections and non detections",
  "name": "magstats",
  "fields": [
    {
      "name": "aid",
      "type": "string"
    },
    {
      "name": "detections",
      "type": {
        "type": "array",
        "items": {
        "type": "record",
        "name": "alert",
        "fields": [
          {
            "name": "oid",
            "type": "string"
          },
          {
            "name": "tid",
            "type": "string"
          },
          {
            "name": "pid",
            "type": "long"
          },
          {
            "name": "candid",
            "type": ["long", "string"]
          },
          {
            "name": "mjd",
            "type": "double"
          },
          {
            "name": "fid",
            "type": "int"
          },
          {
            "name": "ra",
            "type": "double"
          },
          {
            "name": "dec",
            "type": "double"
          },
          {
            "name": "mag",
            "type": "float"
          },
          {
            "name": "e_mag",
            "type": "float"
          },
          {
            "name": "isdiffpos",
            "type": "int"
          },
          {
            "name": "e_ra",
            "type": "float"
          },
          {
            "name": "e_dec",
            "type": "float"
          },
          {
            "name": "extra_fields",
            "type": {
              "default": {},
              "type": "map",
              "values": ["null", "int", "float", "string", "bytes", "boolean"]
            }
          },
          {
            "name": "aid",
            "type": "string"
          },
          {
            "name": "corrected",
            "type": "boolean"
          },
          {
            "name": "dubious",
            "type": "boolean"
          },
          {
            "name": "has_stamp",
            "type": "boolean"
          },
          {
            "name": "mag_corr",
            "type": ["float", "null"]
          },
          {
            "name": "e_mag_corr",
            "type": ["float", "null"]
          },
          {
            "name": "e_mag_corr_ext",
            "type": ["float", "null"]
          }
        ]
      },
        "default": []
      }
    },
    {
      "name": "non_detections",
      "type": {
        "type": "array",
        "items": {
          "type": "record",
          "name": "non_detection",
          "fields": [
            {
              "name": "aid",
              "type": "string"
            },
            {
              "name": "tid",
              "type": "string"
            },
            {
              "name": "oid",
              "type": "string"
            },
            {
              "name": "mjd",
              "type": "double"
            },
            {
              "name": "fid",
              "type": "int"
            },
            {
              "name": "diffmaglim",
              "type": "int"
            }
          ]
        }
      },
      "default": []
    }
  ]
}
```

### Xmatch Step

This step performs a crossmatch with ALLWISE catalog using CDS crossmatch service and sends the result to the xmatch topic of the Kafka server.

#### Input Schema
```python
{
    "doc": "Multi stream light curve with xmatch",
    "name": "alerce.light_curve_xmatched",
    "type": "record",
    "fields": [
        {"name": "aid", "type": "string"},
        {"name": "candid", "type": ["string", "long"]},
        {"name": "meanra", "type": "float"},
        {"name": "meandec", "type": "float"},
        {"name": "ndet", "type": "int"},
        {"name": "detections", "type": DETECTIONS},
        {"name": "non_detections", "type": NON_DETECTIONS},
        {"name": "xmatches", "type": [XMATCH, "null"]},
    ],
}
```

Given the `DETECTIONS` type:

```python
{
    "type": "array",
    "items": {
        "name": "detections_record",
        "type": "record",
        "fields": [
            {"name": "oid", "type": "string"},
            {"name": "tid", "type": "string"},
            {"name": "candid", "type": ["string", "long"]},
            {"name": "mjd", "type": "double"},
            {"name": "fid", "type": "int"},
            {"name": "ra", "type": "double"},
            {"name": "e_ra", "type": "double"},
            {"name": "dec", "type": "double"},
            {"name": "e_dec", "type": "double"},
            {"name": "mag", "type": "float"},
            {"name": "e_mag", "type": "float"},
            {"name": "isdiffpos", "type": "int"},
            {"name": "rb", "type": ["float", "null"]},
            {"name": "rbversion", "type": ["string", "null"]},
            {
                "name": "extra_fields",
                "type": {
                    "type": "map",
                    "values": ["string", "int", "null", "float", "boolean", "double"],
                },
            },
        ],
    },
}
```

the `NON_DETECTIONS` type:


```python
{
    "type": "array",
    "items": {
        "name": "non_detections_record",
        "type": "record",
        "fields": [
            {"name": "tid", "type": "string"},
            {"name": "oid", "type": "string"},
            {"name": "mjd", "type": "double"},
            {"name": "diffmaglim", "type": "float"},
            {"name": "fid", "type": "int"},
        ],
    },
}
```

and the `XMATCH` type:


```python
{
    "type": "map",
    "values": {"type": "map", "values": ["string", "float", "null", "int"]},
}
```
### Features Step

Obtain the features of a given lightcurve, using FeatureExtractors.

#### Input Schema

```python

```

### LC Classifier

Perform a inference given a set of features. This might not get a classification if the object lacks of certain features.

#### Input Schema

```python

```