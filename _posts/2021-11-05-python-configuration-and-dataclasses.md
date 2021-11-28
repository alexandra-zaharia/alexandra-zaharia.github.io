---
title: Python configuration and data classes
date: 2021-11-05 00:00:00 +0100
categories: [Python, configuration]
tags: [python, dataclass, software design]
---

I stumbled upon an interesting [article][best-practices] in which the author describes best practices for working with configuration files in Python. One of the points he makes is that configuration settings should be handled through *identifiers* rather than strings. This is very good advice, since hacking away at a raw dictionary to extract (key, value) pairs is a risky and error-prone endeavor. Spelling mistakes and type errors come to mind.

## Enter data classes

Fortunately, Python 3.7 introduced [`dataclasses`][]. Check [this article][dataclasses-tut] for an in-depth guide. In a nutshell, a data class is a class that essentially holds data (although it can have methods as well), and it comes with mandatory type hints. Data classes are kinda like `struct`s in C++.

Here is how we could encapsulate a simple server configuration using a data class:

```python
from dataclasses import dataclass

@dataclass
class ServerConfig:
    host: str
    port: int
    timeout: float
```

This data class may be instantiated and used as follows:

```python
In [246]: config = ServerConfig('example.com', 80, 0.5)

In [247]: config
Out[247]: ServerConfig(host='example.com', port=80, timeout=0.5)

In [248]: config.host
Out[248]: 'example.com'
```

Data classes are fortunately not limited to attributes. They can have methods, and all the built-in methods including `__init__()` are present. Additionally, there's `__post_init__()` which is used to post-process the instance after `__init__()` is done.

The data class in the previous example can be modified to accept configuration from a dictionary:

```python
@dataclass
class ServerConfigFromDict:
    host: str
    port: int
    timeout: float

    def __init__(self, conf: dict):
        self.host = conf['host']
        self.port = conf['port']
        self.timeout = conf['timeout']
```

If we try to access a non-existing key from the `conf` dict, a `KeyError` is raised. Notice in this example that we do not check whether the value for `'port'` is an int, nor whether the value for `'timeout'` is a float.

This data class may be instantiated as follows:

```python
In [263]: config = ServerConfigFromDict({'host': 'example.com', 'port': 80, 'timeout': 0.5})

In [264]: config
Out[264]: ServerConfigFromDict(host='example.com', port=80, timeout=0.5)

In [265]: config.host
Out[265]: 'example.com'
```

Another solution is to use the [`dacite`][] package:

```python
In [344]: data = {'host': 'example.com', 'port': 80, 'timeout': 0.5}

In [345]: config = dacite.from_dict(data_class=ServerConfig, data=data)

In [346]: config
Out[346]: ServerConfig(host='example.com', port=80, timeout=0.5)

In [347]: config.host
Out[347]: 'example.com'
```


## Dynamically creating a configuration class

The previous examples have shown how to transform a dict-based configuration into a data class. However, we had to make assumptions regarding the configuration itself (what keys it actually contains). There may be situations where you just want to load the whole dict without knowing what keys it contains.

We can do this using the Python [`setattr`][] built-in method. We don't even need a data class for this, a "dumb" class can cut it just as well:

```python
class DynamicConfig:
    def __init__(self, conf):
        if not isinstance(conf, dict):
            raise TypeError(f'dict expected, found {type(conf).__name__}')

        self._raw = conf
        for key, value in self._raw.items():
            setattr(self, key, value)

config = DynamicConfig({'host': 'example.com', 'port': 80, 'timeout': 0.5})
print(f'host: {config.host}, port: {config.port}, timeout: {config.timeout}')
```

Output:

```
host: example.com, port: 80, timeout: 0.5
```

## Dynamic configuration with INI files

The dynamic configuration class we've seen above works well for dicts. However, when parsing certain configuration file formats, the output might be something *dict-like*, i.e. close to but not quite a dict.

Take the INI format for instance. Suppose we have a `config.ini` file with the following contents:

```ini
[server]
host = example.com
port = 80
timeout = 0.5

[user]
username = admin
level = 10
```

INI files may be parsed with [`configparser`][], but the object we get is a `configparser.ConfigParser`. We can create a class to encapsulate such an object and provide identifiers for keys. For the `key = value` part of the INI file, we can reuse our previous `DynamicConfig` class above, but we need to handle the `[sections]` in the INI file separately.

```python
class DynamicConfigIni:
    def __init__(self, conf):
        if not isinstance(conf, configparser.ConfigParser):
            raise TypeError(f'ConfigParser expected, found {type(conf).__name__}')

        self._raw = conf
        for key, value in self._raw.items():
            setattr(self, key, DynamicConfig(dict(value.items())))
```

Here is `DynamicConfigIni` in action:

```python
parser = configparser.ConfigParser()
parser.read_file(open('config.ini'))
config = DynamicConfigIni(parser)
print('server:', config.server.host, config.server.port, config.server.timeout)
print('user:', config.user.username, config.user.level)
```

Output:

```
server: example.com 80 0.5
user: admin 10
```

## Conclusion

In this post we've seen how to encapsulate configuration settings in Python such that we get identifiers instead of strings for the configuration keys. For a simple usage where we know in advance what keys the configuration contains, we can use data classes. For more advanced use cases it is also possible to use `setattr()` to "objectify" the configuration keys. We have looked at a case study for the INI file format, that may be parsed with `configparser`.

Check out [this article][np] for a discussion of passing configuration options in Python through environment variables.

<!-- links -->
[np]: {% post_url 2021-11-25-use-cases-for-python-environment-variables %}
[best-practices]: https://tech.preferred.jp/en/blog/working-with-configuration-in-python/
[`dataclasses`]: https://docs.python.org/3/library/dataclasses.html
[dataclasses-tut]: https://realpython.com/python-data-classes
[`setattr`]: https://docs.python.org/3/library/functions.html#setattr
[`dacite`]: https://pypi.org/project/dacite/0.0.13/
[`configparser`]: https://docs.python.org/3/library/configparser.html