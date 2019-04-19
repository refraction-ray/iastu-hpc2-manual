In this section, some aspects on ansible is discussed.

## Common modules in Ansible

* file
* copy
* command
* debug: `msg` info to be printed
* user
* service
* setup: get basic facts as variables collecting from nodes

## General syntax in playbooks

### Conditionals

* `when`: the argument is rendered by jinja2, but no need for bracket `or`, `and`, `not`
* compare operator in general language is ok to use in the condition statement, linke `!=`, `>=`

### Loops

* `loop`: list or list of hash, corresponding variables in leading task
* some moudules directly support list argument
* register of a loop task, has attr `results` as a list

### Useful keywords

* `environment`: config the env variables, `http_proxy:http://blah`, play level or task level

## CLI commands

* `ansible-vault`, encrypt protected info by given password, used as `!value|eencrypted strings` in playbooks
* `ansible-galaxy init`: create the directory structure for ansible roles
* `ansible-playbook -e "var=value"`, overwrite var with highest priority

## Jinja Template Extension in Ansible

### filters

Jinja template rendering only happen before the task is sent to nodes.

* `int`, string to int


* `to_json`, `to_yaml`, `from_json`, `from_yaml`
* `default(val)` for default value of undefined variables, `default(omit)` use default value from external such as system defaults
* `min`, `flatten`, `flatten(levels=1)`
* `unique`, `union(list)`, `intersect(list)`, `difference(list)`, `symmetric_difference(list)`
* `"{{mac_prefix| random_mac}}"`
* `"{{ 60|random }}"`, `"{{ [1,2,3]|random }}"`
* `"{{ list|shuffle }}"`
* `log(3)`, default base is e,  `pow(5)`, `root(3)`, default power is 2, `abs`, `round`
* `"{{json or yml data| json_query("query pattern")}}"`, this query pattern is backend with [jmespath](http://jmespath.org/)
* `ipaddr`, `ipv4`, `ipv6`, return bool default, but can extract more info if arguments are given, see [doc](https://docs.ansible.com/ansible/latest/user_guide/playbooks_filters_ipaddr.html)
* `parse_cli("spec")` parse output by given spec file, for details of syntax, see [doc](https://docs.ansible.com/ansible/latest/user_guide/playbooks_filters.html#network-cli-filters)
* `hash`, arguments can be sha1 or md5 or more choices. `password_hash('sha256', 'mysecretsalt')`
* `{{dict|combine(dict2)}}` similar to `dict.update(dict2)` in python
* `urlsplit` get info from url by given arguments, see [doc](https://docs.ansible.com/ansible/latest/user_guide/playbooks_filters.html#url-split-filter)
* `regex_search('regexpr')`, `regex_search('regexpr', new)`

### test

Example

```yaml
vars:
  my_var: "blah"
tasks:
  - debug:
      msg: "matched"
      when: my_var is match ("bl*")
```

## Misc

### My comments

In general, ansible playbooks is definitely a type of domain specific language (DSL). There are so many fields, that config files (json or yaml) have been evolving into DSL which go far beyond the scope of some nouns, but also complicated adjective and verbs in the configurations. In some sense, it is always a question, that whether such a scheme is really simple and efficient enough as claimed? Maybe, say playbooks in this case, is easier to handle and suitable for more complicated cases when implemented directly as python scripts instead of a indirect level of abstraction (yml, which one needs reinvent basically all wheels of grammars in a Turing complete language like if and for, and still has less expression power than a Turing complete language). Of course, it is just my personal thought.