# An in-depth guide to the ALeRCE pipeline

Every step works as a "event driven microservice", which listens to a topic and must receive one or more messages in order to perform their operations. The pipeline steps are developed under the [APF Framework](https://github.com/alercebroker/APF). This framework gives some strong guidelines on the structure of the step, in addition of providing some hooks which facilitates the parsing of the information received by the step and the further deployment as well.  

## Pipeline Diagram (detailed)

*insert image here*

## Steps specification

### Sorting Hat

The Sorting Hat is a gateway step which assign an ALeRCE ID (aid) to every incoming alert. This `aid` can be newly generated or an already existing one, depending on the result of the crossmatch performed on the object database. 
At the present time, this step must be deployed with different with different setting to work with the different surveys availible. So you might find two different instances of sorting hat deployed.

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
        {"name": "rb", "type": ["null", "float"]},
        {"name": "rbversion", "type": ["null", "string"]},
        {"name": "mag", "type": "float"},
        {"name": "e_mag", "type": "float"},
        {"name": "rfid", "type": ["null", "int"]},
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

#### Input Schema

### Lightcurve Step 

#### Input Schema

### Correction Step

#### Input Schema

### Magstats Step

#### Input Schema

### Xmatch Step

#### Input Schema

### Features Step

#### Input Schema

### LC Classifier

#### Input Schema