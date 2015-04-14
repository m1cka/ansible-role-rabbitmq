Config Encoder Macros
=====================

Set of Jinja2 and ERB macros which help to encode Python and Ruby data structure
into a different file format.


---

* [Motivation](#motivation)
* [Supported formats](#supported-formats)
* Examples
  * [Ansible example](#ansible-example)
  * [Puppet example](#puppet-example)
* [Macros parameters](#macros-parameters)
* [Limitations](#limitations)
* Programming support
  * [Python example](#python-example)
  * [Ruby example](#ruby-example)
* [License](#license)
* [Author](#author)

---


Motivation
----------

The motivation to create these macros was to be able to create an universal
description of configuration files in [Ansible](http://ansible.com). Ansible and
Puppet is using YAML format as the main data input. This format allows to
describe almost any structure of a configuration file. Ansible resp. Puppet
converts the YAML data into a Python resp. Ruby data structure using primitive
data types (e.g. `dict`, `list`, `string`, `number`, `boolean`, ...). That data
structure is then used in Jinja2 and ERB template files to produce the final
configuration file. The encoder macros are used to convert the Python or Ruby
data structure to another format.


Supported formats
-----------------

- Apache config format
- Erlang config format
- INI
- JSON
- TOML
- XML
- YAML


Ansible example
---------------

Clone this repo into the `templates` directory in the playbook location:

```
$ git clone https://github.com/picotrading/config-encoder-macros ./templates/encoder
```

Then use the macros in your playbook (e.g. `site.yaml`) like this:

```
# Playbook example
- name: Play to test the encoder macros
  hosts: all
  vars:
    ini_data:
      var1: val1
      var2: val2
      section1:
        aaa: asdf
        bbb: 123
        ccc: 'true'
      section2:
        ddd: asdfasd
        eee: 1234
        fff: 'false'
  tasks:
    - name: Create test.ini file
      template:
        src: templates/test.ini.j2
        dest: /tmp/test.ini
```

Where the `templates/test.ini.j2` contains:

```
#
# This file is managed by Ansible.
# Do not edit this file manually.
# Any changes will be automatically reverted.
#

{% from "templates/encoder/macros/ini_encode_macro.j2" import ini_encode with context -%}

{{ ini_encode(ini_data) }}
```

And if you call the playbook:

```
$ ansible-playbook -i hosts ./site.yaml
```

You get the following in the `/tmp/test.ini`:

```
#
# This file is managed by Ansible.
# Do not edit this file manually.
# Any changes will be automatically reverted.
#

var1=val1
var2=val2

[section1]
aaa=asdf
bbb=123
ccc=true

[section2]
ddd=asdfasd
eee=1234
fff=false
```

Look into the `site.yaml` and `templates/*.j2` for more examples.


Puppet example
--------------

Clone this repo to some directory (e.g. `/etc/puppet/encoder`):

```
$ git clone https://github.com/picotrading/config-encoder-macros /etc/puppet/encoder
```

Then use the macros from your Puppet manifest like this:

```
# Load the configuration from YAML file
$ini_data = hiera('ini_data')

file { '/tmp/test.ini' :
  ensure  => present,
  content => template('test.ini.erb'),
}
```

Where the Hiera YAML file contains:

```
---

ini_data:
  var1: val1
  var2: val2
  section1:
    aaa:
      - asdf
      - zxcv
    bbb: 123
    ccc: 'true'
  section2:
    ddd: asdfasd
    eee: 1234
    fff: 'false'
```

And the `templates/test.ini.erb` contains:

```
#
# This file is managed by Puppet.
# Do not edit this file manually.
# Any changes will be automatically reverted.
#

<%-
  item = @ini_data || (ini_data.kind_of?(String) ? eval(ini_data) : ini_data)
-%>
<%= ERB.new(IO.read('/etc/puppet/macros/ini_encode_macro.erb'), nil, '-', '_erbout1').result(OpenStruct.new().send(:binding)) -%>
```

And if you run Puppet you should get the following in the `/tmp/test.ini`:

```
#
# This file is managed by Puppet.
# Do not edit this file manually.
# Any changes will be automatically reverted.
#

var1=val1
var2=val2

[section1]
aaa=asdf
bbb=123
ccc=true

[section2]
ddd=asdfasd
eee=1234
fff=false
```

Look into the `site.pp`, `templates/*.erb` and `puppet_apply.sh` for more
examples.


Macros parameters
-----------------

The first parameter passed to the macro is always the data structure. Any
following parameter should be addressed by a keyword (e.g. `indent="  "` where
`indent` is the keyword).


### `apache_encode()`

- `item`

  > Variable holding the input data for the macro.

- `convert_bools=false`

  > Indicates whether Boolean values presented as a string should be converted
  > to a real Booblean value. For example `var1: 'True'` would be represented
  > as a string but using the `convert_bools=true` will convert it to a Boolean
  > like it would be defined like this: `var1: true`.

- `convert_nums=false`

  > Indicates whether number presented as a string should be converted to
  > number. For example `var1: '123'` would be represented as a string but
  > using the `convert_nums=true` will convert it to a number like it would
  > be defined like this: `var1: 123`. You can also use the YAML type casting
  > to convert string to number (e.g. `!!int "1234"`, `!!float "3.14"`).

- `indent="  "`

  > Defines the indentation unit.

- `type="section"`

  > Defines the type of the `item` in the recursive macro calls. It's used only
  > internally in the macro.

- `level=0`

  > Indicates the initial level of the indentation. Value `0` starts indenting
  > from the beginning of the line. Setting the value to higher than `0`
  > indents the content by `indent * level`.

- `macro_path='macros/apache_encode_macro.erb'`

  > Relative or absolute path to the macro. It's used only in ERB macros.


### `erlang_encode()`

- `item`

  > Variable holding the input data for the macro.

- `convert_bools=false`

  > Indicates whether Boolean values presented as a string should be converted
  > to a real Booblean value. For example `var1: 'True'` would be represented
  > as a string but using the `convert_bools=true` will convert it to a Boolean
  > like it would be defined like this: `var1: true`.

- `convert_nums=false`

  > Indicates whether number presented as a string should be converted to
  > number. For example `var1: '123'` would be represented as a string but
  > using the `convert_nums=true` will convert it to a number like it would
  > be defined like this: `var1: 123`. You can also use the YAML type casting
  > to convert string to number (e.g. `!!int "1234"`, `!!float "3.14"`).

- `first=[]`

  > Indicates whether the item being processed is actually a first item. It's
  > used only internally in the macro.

- `indent="  "`

  > Defines the indentation unit.

- `level=0`

  > Indicates the initial level of the indentation. Value `0` starts indenting
  > from the beginning of the line. Setting the value to higher than `0`
  > indents the content by `indent * level`.

- `macro_path='macros/erlang_encode_macro.erb'`

  > Relative or absolute path to the macro. It's used only in ERB macros.


### `ini_encode()`
- `item`

  > Variable holding the input data for the macro.

- `delimiter="="`

  > Sign separating the *property* and the *value*. By default it's set to `'='`
  > but it can also be set to `' = '`.

- `first=[]`

  > Indicates whether the item being processed is actually a first item. It's
  > used only internally in the macro.

- `macro_path='macros/ini_encode_macro.erb'`

  > Relative or absolute path to the macro. It's used only in ERB macros.

- `quote=""`

  > Allows to set the *value* quoting. Use `quote="'"` or `quote='"'`.

- `section_is_comment=false`

  > Prints sections as comments.

- `ucase_prop=false`

  > Indicates whether the *property* should be made uppercase.


### `json_encode()`
- `item`

  > Variable holding the input data for the macro.

- `convert_bools=false`

  > Indicates whether Boolean values presented as a string should be converted
  > to a real Booblean value. For example `var1: 'True'` would be represented
  > as a string but using the `convert_bools=true` will convert it to a Boolean
  > like it would be defined like this: `var1: true`.

- `convert_nums=false`

  > Indicates whether number presented as a string should be converted to
  > number. For example `var1: '123'` would be represented as a string but
  > using the `convert_nums=true` will convert it to a number like it would
  > be defined like this: `var1: 123`. You can also use the YAML type casting
  > to convert string to number (e.g. `!!int "1234"`, `!!float "3.14"`).

- `indent="  "`

  > Defines the indentation unit.

- `level=0`

  > Indicates the initial level of the indentation. Value `0` starts indenting
  > from the beginning of the line. Setting the value to higher than `0`
  > indents the content by `indent * level`.

- `macro_path='macros/json_encode_macro.erb'`

  > Relative or absolute path to the macro. It's used only in ERB macros.


### `toml_encode()`

- `item`

  > Variable holding the input data for the macro.

- `convert_bools=false`

  > Indicates whether Boolean values presented as a string should be converted
  > to a real Booblean value. For example `var1: 'True'` would be represented
  > as a string but using the `convert_bools=true` will convert it to a Boolean
  > like it would be defined like this: `var1: true`. You can also use the YAML
  > type casting by using `!!bool` in front of your value (e.g.
  > `!!bool "true"`).

- `convert_nums=false`

  > Indicates whether number presented as a string should be converted to
  > number. For example `var1: '123'` would be represented as a string but
  > using the `convert_nums=true` will convert it to a number like it would
  > be defined like this: `var1: 123`. You can also use the YAML type casting
  > to convert string to number (e.g. `!!int "1234"`, `!!float "3.14"`).

- `first=[]`

  > Indicates whether the item being processed is actually a first item. It's
  > used only internally in the macro.

- `indent="  "`

  > Defines the indentation unit.

- `level=0`

  > Indicates the initial level of the indentation. Value `0` starts indenting
  > from the beginning of the line. Setting the value to higher than `0`
  > indents the content by `indent * level`.

- `macro_path='macros/toml_encode_macro.erb'`

  > Relative or absolute path to the macro. It's used only in ERB macros.

- `prevkey=""`

  > Defines the name of the previous key in the recursive macro calls. It's used
  > only internally in the macro.

- `prevtype=""`

  > Defines the type of the previous item in the recursive macro calls. It's used
  > only internally in the macro.

- `quote='"'`

  > Allows to set the *value* quoting. Use `quote="'"` or `quote='"'`.


### `xml_encode()`
- `item`

  > Variable holding the input data for the macro.

- `first=[]`

  > Indicates whether the item being processed is actually a first item. It's
  > used only internally in the macro.

- `indent="  "`

  > Defines the indentation unit.

- `level=0`

  > Indicates the initial level of the indentation. Value `0` starts indenting
  > from the beginning of the line. Setting the value to higher than `0`
  > indents the content by `indent * level`.

- `macro_path='macros/xml_encode_macro.erb'`

  > Relative or absolute path to the macro. It's used only in ERB macros.

- `prevkey=none`

  > Defines the name of the previous key in the recursive macro calls. It's used
  > only internally in the macro.


### `yaml_encode()`
- `item`

  > Variable holding the input data for the macro.

- `convert_bools=false`

  > Indicates whether Boolean values presented as a string should be converted
  > to a real Booblean value. For example `var1: 'True'` would be represented
  > as a string but using the `convert_bools=true` will convert it to a Boolean
  > like it would be defined like this: `var1: true`.

- `convert_nums=false`

  > Indicates whether number presented as a string should be converted to
  > number. For example `var1: '123'` would be represented as a string but
  > using the `convert_nums=true` will convert it to a number like it would
  > be defined like this: `var1: 123`. You can also use the YAML type casting
  > to convert string to number (e.g. `!!int "1234"`, `!!float "3.14"`).

- `indent="  "`

  > Defines the indentation unit.

- `level=0`

  > Indicates the initial level of the indentation. Value `0` starts indenting
  > from the beginning of the line. Setting the value to higher than `0`
  > indents the content by `indent * level`.

- `macro_path='macros/yaml_encode_macro.erb'`

  > Relative or absolute path to the macro. It's used only in ERB macros.

- `quote='"'`

  > Allows to set the *value* quoting. Use `quote="'"` or `quote='"'`.

- `skip_indent=false`

  > Indicates whether the indentation should be skipped for the specific item.
  > It's used only internally in the macro.


Limitations
-----------

### Erlang, JSON, TOML and YAML macros

In these type-sensitive file formats, the Boolean values represented as string
(`"true"`, `"True"`, `"false"` and `"False"`) are automatically converted to
Boolean value (`true` and `false`). The reason for this behavior is that if you
want to pass a Boolean variable in Ansible, you have to actually quote it (e.g.
`var1: "{{ my_boolean_var }}"`). Once the variable is quoted, it's not Boolean
any more. If you want to use Boolean as a string in your final configuration
file, try to use the value with a space (e.g. `' true'`).


### INI macro

This macro doesn't respect the order in which the data was defined. It probably
need to be fixed as some config files might allow to overwrite previous
definitions.


### XML macro

YAML doesn't allow to fully map XML data with attributes. This is why XML
attributes must be defined as part of the element name (e.g. `'elem attr1="val1"
attr2="val2"': Value of the element`). This is also the reason why elements of
the same name can only have the same set of attributes. For example the following YAML
input data:

```
---
root:
  'elem attr1="val1" attr2="val2"':
    - element value1
    - element value2
```

Will produce the following XML file (`elem` has the same set of attributes):

```
<root>
  <elem attr1="val1" attr2="val2">element value1</elem>
  <elem attr1="val1" attr2="val2">element value2</elem>
</root>
```

Another limitation of the XML macro is that attributes can not have value
derivated from a Jinja2 variable in the Ansible context. This macro also doesn't
allow to set order of the elements so it's suitable only for order-insensitive
XML files.


Python example
--------------

Although the macros are intended to be used primarily in Ansible, they can also
be used directly from Python:

```
from jinja2 import Template
from yaml import load
import sys

# Load the YAML data to Python data structure
yaml_fh = open('vars/ini_test.yaml')
yaml_struct = load(yaml_fh)
yaml_fh.close()

# Take only the content of the ini_data variable
yaml_struct = yaml_struct['ini_data']

# Read Jinja2 template as text
template_fh = open('macros/ini_encode_macro.j2')
template_text = template_fh.read()
template_fh.close()

# Create Jinja2 template object
t = Template(template_text)
# Convert the YAML data to INI format
output = t.module.ini_encode(yaml_struct).encode('utf8')

# Print the result
sys.stdout.write(output)
```

Or you can use the `yaml_converter.py`:

```
$ ./yaml_converter.py -f ini -v ini_data -y ./vars/ini_test.yaml
```


Ruby example
------------

```
require 'erb'
require 'ostruct'
require 'yaml'

# Helper variables
yaml_file = './vars/ini_test.yaml'
var = 'ini_data'

# Define variables used in the macro
item = YAML.load_file(yaml_file)[var]
macro_path = './macros/ini_encode_macro.erb'

# Convert the YAML data to the final format
binding = OpenStruct.new.send(:binding)
print ERB.new(IO.read(macro_path), nil, '-', '_erbout0').result(binding)
```

Or you can use the `yaml_converter.rb`:

```
$ ./yaml_converter.rb -f ini -v ini_data -y ./vars/ini_test.yaml
```


License
-------

MIT


Author
------

Jiri Tyr
