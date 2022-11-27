> **Source**: https://whalesalad.com/blog/doing-python-configuration-right

# Doing Python Configuration Right

Let's talk about configuring Python applications, specifically the kind that might live in multiple environments – dev, stage, production, etc...

The tools and frameworks used in the application are not super important because the approach that I will outline below is based on vanilla Python. The impetus for this approach was caused by frustration with [Django's settings](https://docs.djangoproject.com/en/2.2/topics/settings/), but this is my go-to for any kind of Python application I might be working on.

### Recap: Python Modules & Packages

One of my favorite Python features is the way that the files and directories your application is made of map one-to-one with how you import and use them in code.

For example, given this import statement:

```
from app.utils import numbers

```

We can infer the following directory structure:

```
app
├── __init__.py
├── database.py
├── services.py
├── ...
└── utils
    ├── __init__.py
    └── numbers.py

```

Lots of languages and frameworks rely on this novel concept, including [Clojure](https://clojure.org/) and ES6.

In our example, Python considers the `utils` directory a **[Package](https://docs.python.org/3/tutorial/modules.html#packages)**. A directory becomes a package as soon as you place an empty `__init__.py` inside of it.

A common scenario you might encounter as a Python hacker is one where you have a `utils.py` file that eventually gets too large, so you break it out into a `utils/` directory containing many smaller files.

When met with this situation we might do the following:

```
# Create a new package:
mkdir utils && touch utils/__init__.py

# Move our existing code into the new package
mv utils.py utils/something.py

```

So now we've seen that a Python package is decided by the existence of an empty `__init__.py` file in a directory... but **what if the file isn't empty**?

### Putting Code in \_\_init\_\_.py

Since it's just a regular old Python file, you can actually put whatever you want there and it will be executed the first time the package is imported.

You can test this out on your own. Create a directory named `foo` and give it an empty `__init__.py` file.

From a [Python REPL](https://docs.python.org/3/tutorial/interpreter.html) in the same directory:

Seeing no output here is good, it means the statement was successful.

Now let's edit our `__init__.py` file to include the following code:

_`sys.exit()` is typically used to cause a process to [exit with a specific status](http://tldp.org/LDP/abs/html/exitcodes.html)._

Rerunning the same experiment in a new REPL you will observe that **your Python shell immediately exits** after the import. In a larger application the effect would be more pronounced: the whole application would exit.

So we understand the fundamentals and we've seen how this feature can be used maliciously.

Perhaps we can use it for good?

### Multiple Environments & Twelve-Factor Apps

It's likely that your application has lived in multiple environments. Your local **development** environment is likely the first, and you might have a **test** environment that lives on Jenkins or another CI platform. Your code is deployed to a **production** or **live** environment. Some systems might have a **staging** environment that is used just before things go live.

Even if you only consider yourself a hobbyist, developing code locally and deploying it to a VPS or Heroku-like platform means you're dealing with multiple environments.

A rule I follow when building applications is that **I should be able to deploy a codebase – without modification – into any environment**, assuming we have a way to tell the system where it's running.

Contrast this to building multiple artifacts for each deployment target, requiring additional time and complexity to build and persist. These artifacts are typically designed to run in a single target environment, so running them locally or in a test mode is often difficult or impossible.

The famous [twelve-factor methodology](https://12factor.net/config) shares this belief, in addition to the idea that all configuration should exist as environment variables, too. I agree with this to an extent, but there is sometimes a tendency to make _everything_ an environment variable which quickly becomes difficult to support.

If every knob and dial of your system is an environment variable, you'll find that you end up keeping various permutations of variables stored somewhere for running or debugging. See the problem here? We've pulled config out of one area (the code, something that is typically kept in version control) and moved them to an area that is more prone to error and human mistakes.

The general guidelines I use to decide where to draw the line:

- **Static things that don't change often, or things that dramatically influence the behavior of the system should live in the code.**
- **Dynamic things that change frequently, or things that should be kept secret (API keys/credentials) should live outside the code.**

### How do we Switch Environments?

In order for an application to change its behavior between environments we need a way to tell it where it is running. Leaning on environment variables (see a pattern?), I tend to use `ENV` (or a variation) for this purpose.

- The Ruby/Rails ecosystem uses `RACK_ENV` or `RAILS_ENV`
- Javascript projects will oftentimes leverage `NODE_ENV`

I recently completed a project for a client with the following convention:

- My **local development environment** does not set an `ENV` variable, so the system infers `development` by default.
- The **test environment** on AWS CodePipeline uses `ENV=test`
- The **production environment** on EC2 uses `ENV=production`

Note: It's important to consider the consequences of **not** setting this variable. Could that be catastrophic? For instance, could the app boot in DEV mode inside of a production cluster and end up showing tracebacks to the public? For some applications, the default should be production. There is no right or wrong answer here, but it needs to be considered.

### The End Goal

From a developer standpoint we want to access our config like so:

```
from service.config import AWS_S3_BUCKET

```

The import line above doesn't contain anything that would suggest which environment we are in. We don't see the word **development** or **production** anywhere. Instead, we just import what we need and allow the configuration system to decide where that comes from.

**We're leveraging the filesystem and the language itself to provide an API for reading configuration.**

Behind the curtain, this is what the `config` directory looks like on disk:

```
service/config
├── __init__.py
├── common.py
└── environments
   ├── __init__.py
   ├── development.py
   ├── production.py
   └── staging.py

```

- `common.py` contains all of our common or shared configuration. These are things that don't significantly differ from one environment to the next. You could call this `base` or `shared` if you'd like.
- `environments/development.py` contains development configuration. This file **could be excluded from version control** so that each developer on the team can implement his or her own configuration settings.
- `environments/(production|staging).py` include configuration unique to each of their respective environments.

Let's take a look inside `common.py`:

```
import os

APP_NAME = "My Application"

# Conveniences to always have a quick reference to the
# top-level application directory.
ROOT_DIR = os.path.join(
    os.path.dirname(os.path.realpath(__file__)),
    os.pardir,
    os.pardir,
)
SERVICE_DIR = os.path.join(ROOT_DIR, "service")

# Used in DNS lookup jobs.
DEFAULT_NAMESERVERS = [
    "1.1.1.1",
    "1.0.0.1"
]

# Job runner behaviors
JOB_SUPERVISOR_SLEEP_SECONDS = 1
JOB_STATUS_TTL = 10

# AWS Configuration
AWS_DYNAMO_REGION = 'us-west-1'
AWS_DYNAMO_TABLE_PREFIX = 'acme-'
AWS_S3_BUCKET = 'acme-production'

SYSTEM_REBOOT_COMMAND = "sudo systemctl restart foo.service"

```

This is a contrived example so don't read too deeply into the specifics. The important thing to notice is that this is fairly static configuration that is not going to change very often.

Now let's look at `environment/development.py`:

```
from ..common import *

AWS_DYNAMO_TABLE_PREFIX = 'acme-dev-mwhalen-'
AWS_S3_BUCKET = 'acme-dev-mwhalen'

GOOGLE_CLIENT_ID = "XXXX"
GOOGLE_CLIENT_SECRET = "XXXX"
GOOGLE_CLIENT_REFRESH_TOKEN = "XXXX"

# We're inserting our own DNS servers to the front of the defaults.
DEFAULT_NAMESERVERS = [
    "10.0.0.3"
] + DEFAULT_NAMESERVERS

# This is intentionally a no-op command.
# Our specific application is designed to be supervised by systemd,
# but this is not available on macOS.
SYSTEM_REBOOT_COMMAND = "uname -a"

```

- We start by importing the `common` configuration so that we inherit all of the common configuration by default. Now we have the ability to add, replace or augment parameters without the need to copy-paste from the parent.
- To support local development, I can customize the AWS resources being used in my environment. The rest of the system is unchanged, but now my local system is using my own tables in Dynamo as well as my own S3 bucket.
- Because this file is not in version control I can confidently store secrets such as my own `GOOGLE_CLIENT_` credentials.
- Because there is access to the common `DEFAULT_NAMESERVERS` I have the ability extend them versus copy-pasting whatever the common values are into my own configuration.
- In production `systemd` commands are used to restart the application in response to certain admin actions. Because my Mac doesn't have `systemd`, I avoid that problem entirely by replacing the system reboot command with a simple no-op.

### How it Works

Circling back to our `config/__init__.py` file, what could we implement here to make this possible? It's actually quite straightforward:

```
import os
import importlib

# Determine the environment we want to load, default is development.py
ENV = os.environ.get("ENV", "development")

module = importlib.import_module(
    f".environments.{ENV}", package="service.config"
)

# update globals of this module (i.e. settings) with imported.
globals().update(vars(module))


def is_development_env():
    return ENV == "development"


def is_production_env():
    return ENV == "production"

```

We're leveraging **import-time evaluation** to dynamically fetch the necessary configuration from the corresponding child environment. Let's step through it piece by piece:

1.  First we import the `importlib` module ([docs](https://docs.python.org/3/library/importlib.html)) which gives us some handy tools for importing code with code.
2.  Using the convention we established – the `ENV` environment variable – we grab the name of the environment we're currently running in.
3.  We choose `development` as the default if one is not set, but as noted earlier this decision will vary depending on the system.

    We might even consider **preventing our application from starting** unless this variable is defined. Here is an example of how that could work:

    ```
    ENV = os.environ.get("ENV", None)
    if ENV is None:
        raise Exception("The ENV environment variable must be set!")

    ```

4.  Next we use the `importlib.import_module` function to load the module containing our specific environment's code into a local variable, `module`.
5.  Finally, we update the [globals](https://docs.python.org/3/library/functions.html#globals) of this module merging in the ones from the `development.py` file.
6.  At the end you will see a few conveniences (a-la Rails) to make it easier to toggle specific logic based on the environment. These are kept as functions so that they isolate implementation to this module instead of wherever its being used.

This approach was heavily inspired by [Ruby on Rails configuration](https://guides.rubyonrails.org/configuring.html#configuring-rails-components) which achieves a very similar outward appearance albeit with a different under-the-hood implementation.

### A Real World Example

To provide another example of this in action, below is the configuration for this website:

First, here is the exact directory structure of my `config` directory:

```
config
├── __init__.py
├── base.py
└── environments
    ├── __init__.py
    ├── development.py
    ├── production.py
    └── test.py

```

- `development.py` is used locally
- `production.py` is used on Heroku
- `test.py` is used for local unit tests with pytest

```
# config/environments/base.py

import os
import logging
import pytz

LOG_FORMAT = '[%(asctime)s] [%(levelname)s] [%(name)s] %(message)s'

ROOT_DIR = os.path.join(
  os.path.dirname(os.path.realpath(__file__)),
  os.pardir,
  os.pardir
)

SYSTEM_DIR = os.path.join(ROOT_DIR, 'system')
BLOG_DIR = os.path.join(ROOT_DIR, 'blog')
SASS_DIR = os.path.join(ROOT_DIR, 'static', 'sass')

TZ = pytz.timezone('America/Los_Angeles')

def relative_to_root(path):
    return os.path.abspath(os.path.join(ROOT_DIR, path))

DEFAULT_SITE_TITLE = 'Michael Whalen – whalesalad.com'

```

`base.py` contains fairly static configuration:

- A centralized log format to use elsewhere in the project.
- Common directories and a helper function to make path-related work easier.
- The timezone for my service.
- The default title to use when a page doesn't provide it's own.

```
# config/environments/development.py

import os

from ..base import *

DEFAULT_SITE_TITLE = f'[DEV] {DEFAULT_SITE_TITLE}'

REDIS = {
    'host': os.environ.get('REDIS_HOST', 'localhost'),
    'port': os.environ.get('REDIS_PORT', 6379),
    'db': os.environ.get('REDIS_DB', 0),
    'socket_timeout': 120
}

```

In `development.py`, the site title is overridden so while I am editing I know I am looking at a local copy. I also define some local Redis configuration that differs greatly from Production.

```
# config/environments/production.py

import os

from ..base import *

SENTRY_DSN = "https://*******@sentry.io/*******"

REDIS = {
    'url': os.environ.get('REDIS_URL')
}

```

- `SENTRY_DSN` is only defined in `production.py` and not in the base or any other environments. This is to prevent Sentry (centralized error logs) from becoming activated in dev or test situations.
- On Heroku the Redis connection details come from a URL, so that is configured here.

Finally, to demonstrate how this is used elsewhere in the app, take a look at how Redis connections are built:

```
from redis import ConnectionPool
from redis import Redis as R

from system import config


class RedisManager(object):
    @classmethod
    def from_config(cls, redis_config):
        if 'url' in redis_config:
            pool = ConnectionPool.from_url(redis_config['url'])
        else:
            pool = ConnectionPool(**redis_config)

        return cls(connection_pool=pool)

    def __init__(self, connection_pool):
        self.pool = connection_pool

    def get_connection(self):
        return R(connection_pool=self.pool)

    conn = property(get_connection)


Redis = RedisManager.from_config(config.REDIS)

```

Notice the last line: `RedisManager.from_config()` is used to isolate concerns. The rest of `RedisManager` doesn't know what the shape of the data in `config` looks like and shouldn't have to. This is one of the handoff points between the configuration layer and the rest of the system.

### Closing Thoughts

I use this approach in all of my Python projects and have yet to find a situation where this (or a variation of it) doesn't work.

1.  We have the flexibility to create an unlimited number of environments. If for example we wanted to spin-up a temporary environment for a pull request: `cp environments/staging.py environments/PR_402.py` and `ENV=PR_402` is all you need.
2.  When developing locally we can run the system in production mode by prefixing it with `ENV=production` and vice versa, running software anywhere else in a dev or test mode.
3.  Developers can quickly glean the major differences between environments by taking a look at the configuration each of them is overriding. This makes it easier to onboard new team members to your codebase.
4.  Similarly, each developer on the team can have his or her own unique configuration. No more clobbering central config because your system has something setup a little differently than the others.
5.  We can protect our test environment from accidentally reaching out to production resources by explicitly setting certain variables in `environments/test.py` to `None`.
6.  We eliminate the heft of passing big key/val configuration maps between various CLI tools such as Docker et-all (although tooling more and more capable of reading env from a file these days)
7.  We expose our configuration as a vanilla Python package so there is little to no learning curve and interoperability with other Python tools.
8.  We avoid the cost of supporting external libraries/dependencies

At the end of the day this approach is not very glamorous, and that is exactly what we want when we're building systems that need to be reliable, maintainable and efficient. With some plain old Python and a few lines of special code we've unlocked a tremendous amount of flexibility and power in the configuration of our system.

**Did you find this useful? Consider [following @whalesalad on Twitter](https://twitter.com/whalesalad)** so you don't miss out on other techniques for **building better software**.
