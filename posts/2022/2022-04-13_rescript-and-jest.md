---
title: Configuring Jest to work with ReScript
created: 2022-04-13
---

I keep writing blog posts that start out small and baloon until I don't have time for them. The goal here is to write to help me, and hopefully the posts are useful to some others as well. With that in mind, here goes an attempt at a quicker post.

Getting into some ReasonML and OCaml via ReScript (previously BuckleScript). ReScript compiles to JavaScript, allowing code written in it to be runnable however you run JS code - browser, Node.js, whatever you'd like.

Need to be able to write some unit tests, and it turns out that a common JavaScript testing library is easy to use with ReScript - Jest.

# Clone the ReScript template project
Start by cloning the ReScript Project Template. I'm going to rename the cloned copy to `rescript-jest` like so:
```
git clone git@github.com:rescript-lang/rescript-project-template.git rescript-jest
```

Which should leave you with a project roughly like [this](https://github.com/trite/rescript-jest/tree/281ed118a3ea4bd81e5bf4abeaaf8b6c9f47d336).

# Install Jest and ReScript-Jest
```
npm install --save-dev jest
npm install --save-dev @glennsl/rescript-jest
```

After which point your project should look something like [this](https://github.com/trite/rescript-jest/tree/e71d262948db01cc81380f7e5bd744fa076b9e86).

# Jest config initialization
Jest uses a config file for some of its settings. You can use the local NPM tools, or install it globally. I opted for the global install in this case:
```
npm install --global jest
```

You can then run the init command like so, it'll ask a few questions to help create the right settings:
```
PS C:\git\rescript-jest> jest --init

The following questions will help Jest to create a suitable configuration for your project

√ Would you like to use Jest when running "test" script in "package.json"? ... yes
√ Would you like to use Typescript for the configuration file? ... no
√ Choose the test environment that will be used for testing » node
√ Do you want Jest to add coverage reports? ... no
√ Which provider should be used to instrument code for coverage? » v8
√ Automatically clear mock calls, instances and results before every test? ... no

✏️  Modified C:\Git\rescript-jest\package.json

📝  Configuration file created at C:\Git\rescript-jest\jest.config.js
PS C:\git\rescript-jest>
```

[Code at this point](https://github.com/trite/rescript-jest/tree/586a05fac56f9450a7db9f2b0b142db20c01d2c7).

# Some other config changes
Adding another entry in the "scripts" section of package.json makes it easier to check that tests are passing after each file change:
```
"scripts": {
    ...
    "testwatch": "jest --watch"
},
```

Updating the bsconfig.json file like so will allow tests to run properly:
```
"sources": [
    {
        "dir" : "src",
        "subdirs" : true
    },
    {
        "dir": "__tests__",
        "type": "dev"
    }
]
"bs-dev-dependencies": ["@glennsl/rescript-jest"]
```

[Code at this point](https://github.com/trite/rescript-jest/tree/b12983e432e6eccb7aa87dfab5fb5ee448c63c2b)

# Time to test!
Add some code to test to `src/Demo.res`:
```
let doubleSum = (a: int, b: int) => {
    2*a + 2*b
};
```

And then the tests to `__tests__/Demo_test.res`. This test will fail in its current state, which we want to make sure happens:
```
open Jest;

describe("Demo", () => {
    open Expect;

    test("doubleSum 2 inputs", () =>
        expect(Demo.doubleSum(2, 5)) -> toEqual(15))
})
```

Now running a test produces:
```
 FAIL  __tests__/Demo_test.bs.js
  Demo
    × doubleSum 2 inputs (5 ms)

  ● Demo › doubleSum 2 inputs

    expect(received).toEqual(expected) // deep equality

    Expected: 15
    Received: 14

      at app (node_modules/rescript/lib/js/curry.js:14:16)

Test Suites: 1 failed, 1 total
Tests:       1 failed, 1 total
Snapshots:   0 total
Time:        1.007 s, estimated 2 s
Ran all test suites related to changed files.
```

Updating the `toEqual` to a value of 14 should fix it:
```
 PASS  __tests__/Demo_test.bs.js
  Demo
    √ doubleSum 2 inputs (3 ms)

Test Suites: 1 passed, 1 total
Tests:       1 passed, 1 total
Snapshots:   0 total
Time:        0.278 s, estimated 1 s
Ran all test suites.
```

You should hopefully be up and running now, code at this point might look like [this](https://github.com/trite/rescript-jest/tree/7b95493c1ca53049063fe8cf2fa25f70a0359167).

# References

https://jestjs.io/docs/getting-started

https://github.com/glennsl/rescript-jest
