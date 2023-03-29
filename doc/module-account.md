# Account Module

> "No sensible decision can be made any  
>  longer without taking into account  
>  not only the world as it is, but  
>  the world as it will be."  
>  --- Isaac Asimov

The SCE's account request interface allows the user to determine the number and
name of each *account* set up for the SCE in a given location. An account is
defined as a set of setup files associated with an account number which starts
at 1 and has a maximum value of 9,999. The setup files for a given account
number configure the SCE to compute loans with protection in a particular
manner.

The `clients.set` file (located in the same `data/` directory
as the other setup files) holds the name of each setup file. These names
are user defined, and may be edited by you using any text editor. The
first account name (corresponding to account #0001) is on the *second*
line, with subsequent account names following on separate lines.

## Account Request Data Object Field Definition

**Example - Request Envelope for Account Module**

```json
{
  "Module" : "Account",
  "Data" : {
    "DataPath" : "/etc/sce/",
    "Begin" : "1",
    "End" : "9999"
  }
}
```

The fields of the `Data` object supported by an Account module request are defined
in alphabetical order below:

### 🟦 Begin

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | Number - Integer | 1 |

The `Begin` field specifies the desired starting account number to return
information on. If the value of the `Begin` field exceeds the number of accounts
defined in a given `clients.set` file, then a value of `1` will be used.

### 🟦 DataPath

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | Text | default data path |

If this field is set, the SCE will look for a `/data` folder containing the
setup files in the path specified. Thus, if the `DataPath` is set to `/etc/sce`
(as above), the SCE will look for the setup files in `/etc/sce/data`.

If the calling application wishes to specify the data directory path in its
entirety (e.g. the calling app does not want the SCE to append `/data` to the
provided path), then simply terminate the specified `DataPath` with an asterisk
(`*`). Thus, if the `DataPath` is set to `/etc/sce/bank1/*`, the SCE will look for
the setup files in `/etc/sce/bank1/`.

If this field is not set, the SCE will attempt to located the `/data` folder in
the default data directory path location, which can be retrieved using the
`Version` module.

This field is useful if you wish to use only a single installation of the SCE,
but have many different setup file groupings. By specifying a different
`DataPath` for each grouping, you can easily separate the groups from one
another instead of grouping them all together in a single directory.

### 🟦 End

| Type  | Required | Values | Default |
| :---: |   :---:  |  ---   |  :---:  |
| String | no | Number - Integer | 9999 |

The `End` field specifies the desired ending account number to return
information on. If the value of the `End` field exceeds the number of accounts
defined in a given `clients.set` file, then the number of accounts will be used.

## Account Response Data Object Field Definition

**Example - Response Envelope for Account Module**

```json
{
  "Result" : 200,
  "Module" : "Account",
  "Data" : {
    "Errors" : [
    ],
    "Warnings" : [
    ],
    "DataPath" : "/etc/sce/data/",
    "Accounts" : [
      {
        "Account" : 1,
        "Title" : "Single Premium"
      },
      {
        "Account" : 2,
        "Title" : "MOB"
      },
      {
        "Account" : 3,
        "Title" : "True MOB"
      }
    ]
  }
}
```

The `Data` object for a response from the Account module is defined below:

### 🟥 Errors

| Type  | Required |
| :---: |   :---:  |
| array of String | yes |


The `Errors[]` field contains an array of Strings which describe any errors encountered
while handling the request. If the length of the `Errors[]` Array is zero (0), then the
module processed the request successfully, and the Data object can be further processed
by the calling application for the returned data.

On the other hand, if the length of the `Errors[]` Array is greater than zero (0), then
this indicates that an error condition has been detected, and the calling application
should *not* process the respons Data object further. In this case, the contents of the
`Errors[]` array will describe the error(s) encountered.

Typical errors include the omission of *🟥 required* fields, invalid field values, etc.

### 🟥 Warnings

| Type  | Required |
| :---: |   :---:  |
| array of String | yes |


The `Warnings[]` field contains an array of Strings which describe any warnings generated
by the module handling the request. The most common warnings returned by modules inform
the calling application that the module does not recognize a specified field (which may
help to isolate a field name spelling error in the calling application's code). Note that
field names which start with "//" will bre treated as comment fields by the SCEJSON, and
no warnings will be generated for these unrecognized fields.

**Example - Request and response illustrating warnings when passing unrecognized fields**
```json
{
  "Module" : "Account",
  "Data" : {
    "//" : "This is a comment.",
    "Hello" : "Friend!",
    "How" : "are you?"
  }
}
```

```json
{
  "Result" : 200,
  "Module" : "Account",
  "Data" : {
    "Errors" : [
    ],
    "Warnings" : [
      "Request field Data.Hello (String) not recognized.",
      "Request field Data.How (String) not recognized."
    ],
    "DataPath" : "/home/alank/Work/sce/scejson/data/",
    "Accounts" : [
      {
        "Account" : 1,
        "Title" : "Single Premium"
      },
      {
        "Account" : 2,
        "Title" : "MOB"
      },
      {
        "Account" : 3,
        "Title" : "True MOB"
      }
    ]
  }
}   
```

### 🟥 DataPath

| Type  | Required |
| :---: |   :---:  |
| String | yes |

The fully qualified directory name used by the SCE to find the `clients.set`
file. If the SCE can not find the `clients.set` file in the data directory,
or if the data direcotry does not exist, then an error will be returned.

### 🟥 Accounts

| Type  | Required |
| :---: |   :---:  |
| array of Account objects | yes |

The `Accounts[]` array returns information on each account found in the
`clients.set` file.

<details>
<summary><b>Account fields</b></summary>

---

🟥 **Account.Account**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Number - Integer |

Discloses the account number associated with this account.

🟥 **Account.Title**

| Type  | Required | Values |
| :---: |   :---:  |  ---   |
| String | yes | Text |

Holds the title of this account.

---

</details>

| ⬅️ Back | ⬆️ Up | Forward ➡️ |
| :--- | :---: | ---: |
| [API Overview](api-overview.md) | [SCEJSON Reference Manual](README.md) | [User Interface](module-ui.md) |
