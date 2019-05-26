In this section, some aspects on ansible is discussed.

## Common modules in Ansible

* file
* copy
* command
* cron: name attr must be specfied, otherwise, the crontab task shall be added repeatedly
* debug: `msg` info to be printed
* user
* service
* setup: get basic facts as variables collecting from nodes
* template: backup=yes
* authorized_keys
* [git](https://docs.ansible.com/ansible/latest/modules/git_module.html)
* lineinfile
* apt
* make
* hostname
* [mysql_user](https://docs.ansible.com/ansible/latest/modules/mysql_user_module.html): mysql-python package missing issue: [post](https://github.com/geerlingguy/ansible-role-mysql/issues/42), `apt: name=python3-mysqldb state=present` is enough, however `pip install mysql-python` wouldn't work.
* pip

## General syntax in playbooks

### Conditionals

* `when`: the argument is rendered by jinja2, but no need for bracket `or`, `and`, `not`
* compare operator in general language is ok to use in the condition statement, linke `!=`, `>=`
* `changed_when`, `failed_when`

### Loops

* `loop`: list or list of hash, corresponding variables in leading task
* some moudules directly support list argument
* register of a loop task, has attr `results` as a list
* `with_items`, register.results is automatically a list, see [this post](https://stackoverflow.com/questions/29512443/register-variables-in-with-items-loop-in-ansible-playbook/29564339)

### Useful keywords

* `environment`: config the env variables, `http_proxy:http://blah`, play level or task level
* `become`: the user after ssh
* `remote_user`: the user used for ssh

## CLI commands

* `ansible-vault`, encrypt protected info by given password, used as `!value|eencrypted strings` in playbooks
* `ansible-galaxy init`: create the directory structure for ansible roles
* `ansible-playbook -e "var=value"`, overwrite var with highest priority
* `-vv`: verbose mode
* example: `ansible -i ~/hpc/hosts cn -m apt -a "name=iperf state=present" --become -K`

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

## Ansible Vault

* [doc](https://ansible-tran.readthedocs.io/en/latest/docs/playbooks_vault.html)

## Development consideration

* [practical example of modules and action plugins](https://ndemengel.github.io/2015/01/20/ansible-modules-and-action-plugins/)
* Future plan: a spack module for ansible (settled by `changed_when` to avoid changed report of command)

## Misc

### Cautions

* In template system, just use `{{}}` instead of quote `""` outside.

* indent in jinja template config files: [blog](https://tech.just-imho.net/2016/06/09/ansible-indenting-in-templates/)

* `ansible_facts`, the key should rip the ansible part off, which is ...

* but for `host_vars[hostname]` to access the facts, the ansible prefix is must, which is in contrast with `ansible_facts`...

* lookup plugin dont take `become: yes` as a thing, it just cannot cat other user's file….

* for `copy` to copy files without permission, use remote_src: yes option, otherwise `become` is also useless...

* the dest path cannot be a relative one, but use `{{role_path}}` instead

* each task has it own ssh session and shell: [how source work with ansible](https://stackoverflow.com/questions/22256884/not-possible-to-source-bashrc-with-ansible/27541856#27541856). The default shell of ansible is `sh`, while source is a bash builtin instead of sh.

* `{{ D['key']|default ('undefined') }}` can be used as default value for non existing keys of dict

### My comments

In general, ansible playbooks is definitely a type of domain specific language (DSL). There are so many fields, that config files (json or yaml) have been evolving into DSL which go far beyond the scope of some nouns, but also complicated adjective and verbs in the configurations. In some sense, it is always a question, that whether such a scheme is really simple and efficient enough as claimed? Maybe, say playbooks in this case, is easier to handle and suitable for more complicated cases when implemented directly as python scripts instead of a indirect level of abstraction (yml, which one needs reinvent basically all wheels of grammars in a Turing complete language like if and for, and still has less expression power than a Turing complete language). Of course, it is just my personal thought.