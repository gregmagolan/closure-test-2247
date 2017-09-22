## Test for issue https://github.com/google/closure-compiler/issues/2247

Build still fails with latest google-closure-compiler 20170910.0.0, although the failures are different than the version used in the issue (20161201.0.0).

### Usage

* `yarn install`
* `yarn build`
* `yarn start`

### Tests with Module resolution BROWSER

Using the default module resolution `BROWSER`, the failure is:

```
stripme/rxjs/add_first.js:1: WARNING - Failed to load module "./Observable"
import { Observable } from './Observable';
^

stripme/rxjs/add_first.js:2: WARNING - Failed to load module "./operator_first"
import { first } from './operator_first';
^

user.js:1: WARNING - Invalid module path "rxjs/add_first" for resolution mode "BROWSER"
import 'rxjs/add_first';
^

user.js:1: ERROR - required "module$rxjs$add_first" namespace not provided yet
import 'rxjs/add_first';
^^^^^^^^^^^^^^^^^^^^^^^^

1 error(s), 3 warning(s)
```

Changing the `rxjs/add_first` import to relative `./rxjs/add_first`, the failure becomes similar to what is described in the issue:

```
stripme/rxjs/add_first.js:1: WARNING - Failed to load module "./Observable"
import { Observable } from './Observable';
^

stripme/rxjs/add_first.js:2: WARNING - Failed to load module "./operator_first"
import { first } from './operator_first';
^

user.js:1: WARNING - Failed to load module "./rxjs/add_first"
import './rxjs/add_first';
^

user.js:1: ERROR - required "module$rxjs$add_first" namespace not provided yet
import './rxjs/add_first';
^^^^^^^^^^^^^^^^^^^^^^^^^^

1 error(s), 3 warning(s)
```

Adding `export var a;` to `add_first.js` results in the same failure.

### Tests with Module resolution NODE

Switching to `--module_resolution=NODE` and back to an ambigious import path `rxjs/add-first`, the failure is now:

```
user.js:1: WARNING - Failed to load module "rxjs/add_first"
import 'rxjs/add_first';
^

user.js:1: ERROR - required "module$rxjs$add_first" namespace not provided yet
import 'rxjs/add_first';
^^^^^^^^^^^^^^^^^^^^^^^^

1 error(s), 1 warning(s)
```

With module_resolution NODE and a relative import path `./rxjs/add-first`, the failure is:

```
user.js:1: ERROR - required "module$rxjs$add_first" namespace not provided yet
import './rxjs/add_first';
^^^^^^^^^^^^^^^^^^^^^^^^^^

1 error(s), 0 warning(s)
```

Re-ordering the --js files in closure conf to:

```
--js stripme/rxjs/*.js
--js user.js
```

The build is now successful & runs (but only with a relative import path in `./rxjs/add-first`; with an absolute import path `rxjs/add-first` you still get the error `user.js:1: WARNING - Failed to load module "rxjs/add_first"`).

Build output is 454 bytes.

### Succesfully build with module resolution NODE with patches applied

Pulling in my compiler dist at https://github.com/gregmagolan/closure-compiler.git#20170919.angular.dist (HEAD on 20170919 + patches) and running the build with `module_resolution=NODE`, the build succeeds and runs in all variations (build output is consistently 454 bytes).

* with relative import `./rxjs/add-first`
* with ambigous import `rxjs/add-first`
* with different `--js` orders in closure.conf:

```
--js user.js
--js stripme/rxjs/*.js
```

and

```
--js stripme/rxjs/*.js
--js user.js
```

* no `export var a;` is needed
