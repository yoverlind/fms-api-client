<!--@h1([pkg.name])-->
# fms-api-client
<!--/@-->

[![Build Status](https://travis-ci.org/Luidog/fms-api-client.png?branch=master)](https://travis-ci.org/Luidog/fms-api-client) [![Known Vulnerabilities](https://snyk.io/test/github/Luidog/fms-api-client/badge.svg?targetFile=package.json)](https://snyk.io/test/github/Luidog/fms-api-client?targetFile=package.json) [![Coverage Status](https://coveralls.io/repos/github/Luidog/fms-api-client/badge.svg?branch=master)](https://coveralls.io/github/Luidog/fms-api-client?branch=master) [![GitHub issues](https://img.shields.io/github/issues/Luidog/fms-api-client.svg?style=plastic)](https://github.com/Luidog/fms-api-client/issues) [![Github commits (since latest release)](https://img.shields.io/github/commits-since/luidog/fms-api-client/latest.svg)](https://img.shields.io/github/issues/Luidog/fms-api-client.svg)  [![GitHub license](https://img.shields.io/github/license/Luidog/fms-api-client.svg)](https://github.com/Luidog/fms-api-client/blob/master/LICENSE.md)

A FileMaker Data API client designed to allow easier interaction with a FileMaker application from a web environment. This client abstracts the FileMaker 17 Data API into class based methods. You can find detailed documentation on this project here:

[fms-api-client documentation]: https://luidog.github.io/fms-api-client/

<!--@installation()-->
## Installation

```sh
npm install --save fms-api-client
```
<!--/@-->

## Usage

```js
/**
 * Connect must be called before the filemaker class is instiantiated. This
 * connect uses Marpat. Marpat is a fork of Camo. much love to
 * https://github.com/scottwrobinson for his creation and maintenance of Camo.
 * My fork of Camo - Marpat is designed to allow the use of multiple datastores
 * with the focus on encrypted storage. Find out more about [fms-api-client documentation]: https://luidog.github.io/fms-api-client/

const environment = require('dotenv');
const varium = require('varium');
const { connect } = require('marpat');
const { Filemaker } = require('fms-api-client');

environment.config({ path: './tests/.env' });

varium(process.env, './tests/env.manifest');



connect('nedb://memory').then(db => {
  /**
   * The client is the FileMaker class. The class then offers methods designed to
   * make it easier to integrate into filemaker's api.
   */

  const client = Filemaker.create({
    application: process.env.APPLICATION,
    server: process.env.SERVER,
    user: process.env.USERNAME,
    password: process.env.PASSWORD
  });

  /**
   * A client can be used directly after saving it. It is also stored on the
   * datastore so that it can be reused later.
   */
  client.save().then(client =>
    /**
     * Using the client you can create filemaker records. To create a record
     * specify the layout to use and the data to insert on creation. The client
     * will automatically convert numbers, arrays, and objects into strings so
     * they can be inserted into a filemaker field.
     */
    client
      .create(
        'Heroes',
        {
          name: 'George Lucas',
          number: 5,
          array: ['1'],
          object: { driods: true }
        },
        { merge: true }
      )
      .then(record =>
        console.log('Some guy thought of a movie....'.yellow.underline, record)
      )
      .catch(error => console.log('That is no moon....'.red, error))
  );

  /**
   * Most methods on the client are promises. The only exceptions to this are
   * the utility methods of fieldData(), and recordId(). You can chain together
   * multiple methods such as record creation.
   */
  client
    .save()
    .then(client => {
      return Promise.all([
        client.create('Heroes', { name: 'Anakin Skywalker' }),
        client.create('Heroes', { name: 'Obi-Wan' }),
        client.create('Heroes', { name: 'Yoda' })
      ]).then(response => {
        console.log('A Long Time Ago....'.rainbow.underline, response);
        return client;
      });
    })
    .then(client => {
      /**
       * You can use the client to list filemaker records. The List method
       * accepts a layout and parameter variable. The client will automatically
       * santize the limit, offset, and sort keys to correspond with the Data
       * API's requirements.
       */
      client
        .list('Heroes', { limit: 5 })
        .then(response => client.fieldData(response.data))
        .then(response =>
          console.log(
            ' For my ally is the Force, and a powerful ally it is.'.underline
              .green,
            response
          )
        )
        .catch(error => console.log('That is no moon....'.red, error));
      /**
       * You can also use the client to set FileMaker Globals for the session.
       */
      client
        .globals({ 'Globals::ship': 'Millenium Falcon' })
        .then(response =>
          console.log(
            'Made the Kessel Run in less than twelve parsecs.'.underline.blue,
            response
          )
        )
        .catch(error => console.log('That is no moon....'.red, error));

      return client;
    })
    .then(client => {
      /**
       * The client's find method  will accept either a single object as find
       * parameters or an array. The find method will also santize the limit,
       * sort, and offset parameters to conform with the Data API's
       * requirements.
       */
      client
        .find('Heroes', [{ name: 'Anakin Skywalker' }], { limit: 1 })
        .then(response => client.recordId(response.data))
        .then(recordIds =>
          client.edit('Heroes', recordIds[0], { name: 'Darth Vader' })
        )
        .then(response =>
          console.log(
            'I find your lack of faith disturbing'.cyan.underline,
            response
          )
        )
        .catch(error => console.log('That is no moon...'.red, error));

      client
        .upload('./assets/placeholder.md', 'Heroes', 'image')
        .then(response => {
          console.log('Perhaps an Image...'.cyan.underline, response);
        })
        .catch(error => console.log('That is no moon...'.red, error));

      client
        .find('Heroes', [{ name: 'Luke Skywalker' }], { limit: 1 })
        .then(response => client.recordId(response.data))
        .then(recordIds =>
          client.upload(
            './assets/placeholder.md',
            'Heroes',
            'image',
            recordIds[0]
          )
        )
        .then(response => {
          console.log('Dont Forget Luke...'.cyan.underline, response);
        })
        .catch(error => console.log('That is no moon...'.red, error));

      client
        .script('FMS Triggered Script', 'Heroes')
        .then(response => {
          console.log('or a script....'.cyan.underline, response);
        })
        .catch(error => console.log('That is no moon...'.red, error));
    })
    .catch(error => console.log('That is no moon...'.red, error));

  client
    .find('Heroes', [{ name: 'Darth Vader' }], {
      limit: 1,
      script: 'example script',
      'script.param': 'han'
    })
    .then(response =>
      console.log(
        'I find your lack of faith disturbing'.cyan.underline,
        response
      )
    )
    .catch(error => console.log('find - That is no moon...'.red, error));
});

const rewind = () => {
  Filemaker.findOne().then(client => {
    console.log(client.data.status());
    client
      .find('Heroes', [{ id: '*' }], { limit: 10 })
      .then(response => client.recordId(response.data))
      .then(response => {
        console.log('Be Kind.... Rewind.....'.rainbow, response);
        return response;
      })
      .then(recordIds =>
        recordIds.forEach(id => {
          client
            .delete('Heroes', id)
            .catch(error => console.log('That is no moon....'.red, error));
        })
      );
  });
};

setTimeout(() => rewind(), 10000);
```

## Tests

```sh
npm install
npm test
```

<!--@execute('npm run test',[])-->
```default
> fms-api-client@1.4.0 test /fms-api-client
> nyc _mocha --recursive  ./tests --timeout=30000 --exit



  Authentication Capabilities
    ✓ should authenticate into FileMaker. (149ms)
    ✓ should automatically request an authentication token (172ms)
    ✓ should reuse a saved authentication token (171ms)
    ✓ should log out of the filemaker. (164ms)
    ✓ should not attempt a logout if there is no valid token.
    ✓ should reject if the logout request fails (172ms)
    ✓ should reject if the authentication request fails (1419ms)

  Create Capabilities
    ✓ should create FileMaker records. (172ms)
    ✓ should reject bad data with an error (167ms)
    ✓ should create FileMaker records with mixed types (168ms)
    ✓ should substitute an empty object if data is not provided (159ms)
    ✓ should return an object with merged filemaker and data properties (164ms)
    ✓ should allow you to run a script when creating a record with a merge response (181ms)
    ✓ should allow you to specify scripts as an array (178ms)
    ✓ should allow you to specify scripts as an array with a merge response (177ms)
    ✓ should sanitize parameters when creating a new record (178ms)
    ✓ should accept both the default script parameters and a scripts array (179ms)

  Delete Capabilities
    ✓ should delete FileMaker records. (239ms)
    ✓ should trigger scripts via an array when deleting records. (247ms)
    ✓ should trigger scripts via parameters when deleting records. (244ms)
    ✓ should allow you to mix script parameters and scripts array when deleting records. (246ms)
    ✓ should stringify script parameters. (245ms)
    ✓ should reject deletions that do not specify a recordId (169ms)
    ✓ should reject deletions that do not specify an invalid recordId (166ms)

  Edit Capabilities
    ✓ should edit FileMaker records.
    ✓ should reject bad data with an error (257ms)
    ✓ should return an object with merged filemaker and data properties
    ✓ should allow you to run a script when editing a record (253ms)
    ✓ should allow you to run a script via a scripts array when editing a record (261ms)
    ✓ should allow you to specify scripts as an array (207ms)
    ✓ should allow you to specify scripts as an array with a merge response (185ms)
    ✓ should sanitize parameters when creating a new record (186ms)
    ✓ should accept both the default script parameters and a scripts array (179ms)

  Find Capabilities
    ✓ should perform a find request (255ms)
    ✓ should allow you to use an object instead of an array for a find (201ms)
    ✓ should specify omit Criterea (249ms)
    ✓ should allow additional parameters to manipulate the results (167ms)
    ✓ should allow you to limit the number of portal records to return (176ms)
    ✓ should allow you to use numbers in the find query parameters (167ms)
    ✓ should allow you to sort the results (185ms)
    ✓ should return an empty array if the find does not return results (173ms)
    ✓ should allow you run a pre request script (169ms)
    ✓ should return a response even if a script fails (179ms)
    ✓ should allow you to send a parameter to the pre request script (178ms)
    ✓ should allow you run script after the find and before the sort (201ms)
    ✓ should allow you to pass a parameter to a script after the find and before the sort (202ms)
    ✓ should reject of there is an issue with the find request (164ms)

  Get Capabilities
    ✓ should get specific FileMaker records. (243ms)
    ✓ should reject get requests that do not specify a recordId (238ms)
    ✓ should allow you to limit the number of portal records to return (235ms)
    ✓ should accept namespaced portal limit and offset parameters (240ms)

  Global Capabilities
    ✓ should allow you to set FileMaker globals (163ms)
    ✓ should reject with a message and code if it fails to set a global (165ms)

  List Capabilities
    ✓ should allow you to list records (253ms)
    ✓ should allow you use parameters to modify the list response (169ms)
    ✓ should should allow you to use numbers in parameters (170ms)
    ✓ should should allow you to provide an array of portals in parameters (213ms)
    ✓ should should remove non used properties from a portal object (174ms)
    ✓ should modify requests to comply with DAPI name reservations (174ms)
    ✓ should allow strings while complying with DAPI name reservations (174ms)
    ✓ should allow you to offset the list response (183ms)
    ✓ should santize parameters that would cause unexpected parameters (173ms)
    ✓ should allow you to limit the number of portal records to return (177ms)
    ✓ should accept namespaced portal limit and offset parameters (170ms)
    ✓ should reject invalid parameters (172ms)

  Script Capabilities
    ✓ should allow you to trigger a script in FileMaker (183ms)
    ✓ should allow you to trigger a script in FileMaker (177ms)
    ✓ should allow you to trigger a script in a find (254ms)
    ✓ should allow you to trigger a script in a list (173ms)
    ✓ should allow reject a script that does not exist (172ms)
    ✓ should allow return a result even if a script returns an error (185ms)
    ✓ should parse script results if the results are json (194ms)
    ✓ should not parse script results if the results are not json (169ms)
    ✓ should parse an array of scripts (188ms)
    ✓ should trigger scripts on all three script phases (190ms)

  Storage
    ✓ should allow an instance to be created
    ✓ should allow an instance to be saved.
    ✓ should allow an instance to be recalled
    ✓ should allow insances to be listed
    ✓ should allow you to remove an instance

  File Upload Capabilities
    ✓ should allow you to upload a file to a new record (1355ms)
    ✓ should allow you to upload a file to a specific container repetition (1345ms)
    ✓ should reject with a message if it can not find the file to upload (164ms)
    ✓ should allow you to upload a file to a specific record (1352ms)
    ✓ should allow you to upload a file to a specific record container repetition (1535ms)
    ✓ should reject of the request is invalid (313ms)

  Data Usage Tracking Capabilities
    ✓ should track API usage data. (168ms)
    ✓ should allow you to reset usage data. (168ms)

  Utility Capabilities
    ✓ should extract field while maintaining the array (242ms)
    ✓ should extract field data while maintaining the object (247ms)
    ✓ should extract the recordId while maintaining the array (247ms)
    ✓ should extract field data while maintaining the object (243ms)


  92 passing (23s)

-----------------------|----------|----------|----------|----------|-------------------|
File                   |  % Stmts | % Branch |  % Funcs |  % Lines | Uncovered Line #s |
-----------------------|----------|----------|----------|----------|-------------------|
All files              |      100 |      100 |      100 |      100 |                   |
 fms-api-client        |      100 |      100 |      100 |      100 |                   |
  index.js             |      100 |      100 |      100 |      100 |                   |
 fms-api-client/src    |      100 |      100 |      100 |      100 |                   |
  client.model.js      |      100 |      100 |      100 |      100 |                   |
  connection.model.js  |      100 |      100 |      100 |      100 |                   |
  credentials.model.js |      100 |      100 |      100 |      100 |                   |
  data.model.js        |      100 |      100 |      100 |      100 |                   |
  index.js             |      100 |      100 |      100 |      100 |                   |
  utilities.service.js |      100 |      100 |      100 |      100 |                   |
-----------------------|----------|----------|----------|----------|-------------------|
```
<!--/@-->

<!--@dependencies()-->
## <a name="dependencies">Dependencies</a>

- [axios](https://github.com/axios/axios): Promise based HTTP client for the browser and node.js
- [form-data](https://github.com/form-data/form-data): A library to create readable "multipart/form-data" streams. Can be used to submit forms and file uploads to other web applications.
- [lodash](https://github.com/lodash/lodash): Lodash modular utilities.
- [marpat](https://github.com/luidog/marpat): A class-based ES6 ODM for Mongo-like databases.
- [moment](https://github.com/moment/moment): Parse, validate, manipulate, and display dates
- [object-sizeof](https://github.com/miktam/sizeof): Sizeof of a JavaScript object in Bytes
- [prettysize](https://github.com/davglass/prettysize): Convert bytes to other sizes for prettier logging

<!--/@-->

<!--@devDependencies()-->
## <a name="dev-dependencies">Dev Dependencies</a>

- [chai](https://github.com/chaijs/chai): BDD/TDD assertion library for node.js and the browser. Test framework agnostic.
- [chai-as-promised](https://github.com/domenic/chai-as-promised): Extends Chai with assertions about promises.
- [colors](https://github.com/Marak/colors.js): get colors in your node.js console
- [coveralls](https://github.com/nickmerwin/node-coveralls): takes json-cov output into stdin and POSTs to coveralls.io
- [dotenv](https://github.com/motdotla/dotenv): Loads environment variables from .env file
- [eslint](https://github.com/eslint/eslint): An AST-based pattern checker for JavaScript.
- [eslint-config-google](https://github.com/google/eslint-config-google): ESLint shareable config for the Google style
- [eslint-config-prettier](https://github.com/prettier/eslint-config-prettier): Turns off all rules that are unnecessary or might conflict with Prettier.
- [eslint-plugin-prettier](https://github.com/prettier/eslint-plugin-prettier): Runs prettier as an eslint rule
- [jsdocs](https://github.com/xudafeng/jsdocs): jsdocs
- [minami](https://github.com/Nijikokun/minami): Clean and minimal JSDoc 3 Template / Theme
- [mocha](https://github.com/mochajs/mocha): simple, flexible, fun test framework
- [mocha-lcov-reporter](https://github.com/StevenLooman/mocha-lcov-reporter): LCOV reporter for Mocha
- [mos](https://github.com/mosjs/mos): A pluggable module that injects content into your markdown files via hidden JavaScript snippets
- [mos-plugin-dependencies](https://github.com/mosjs/mos/tree/master/packages/mos-plugin-dependencies): A mos plugin that creates dependencies sections
- [mos-plugin-execute](https://github.com/team-767/mos-plugin-execute): Mos plugin to inline a process output
- [mos-plugin-installation](https://github.com/mosjs/mos/tree/master/packages/mos-plugin-installation): A mos plugin for creating installation section
- [mos-plugin-license](https://github.com/mosjs/mos-plugin-license): A mos plugin for generating a license section
- [mos-plugin-snippet](https://github.com/mosjs/mos/tree/master/packages/mos-plugin-snippet): A mos plugin for embedding snippets from files
- [nyc](https://github.com/istanbuljs/nyc): the Istanbul command line interface
- [prettier](https://github.com/prettier/prettier): Prettier is an opinionated code formatter
- [varium](https://npmjs.org/package/varium): A strict parser and validator of environment config variables

<!--/@-->

<!--@license()-->
## License

MIT © Lui de la Parra
<!--/@-->
