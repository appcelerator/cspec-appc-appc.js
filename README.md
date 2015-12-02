# CSPEC 2 - app.js unified metadata file

This [CSPEC](https://github.com/appcelerator/cspec) is for standardizing around a unified metadata file named `appc.js`.

## Audience

The primary audience for this proposal is any developer interacting with Appcelerator products.

## Goals

The goals are to come up with a unified file which can describe the project in a way that it can be tooled or processed by systems to understand metadata about the project.  The file should be simple enough for an end user developer to be able to edit it.

## Description

The problem is we have multiple types of files that do this across different products or none at all.  Titanium uses tiapp.xml, Alloy uses alloy.json, Arrow uses appc.json, etc.  To intention is to come up with a universal file which can support any product and provide enough metadata such that the project can be easily understood by tools.

## Proposal

The proposal would be to standardize on a file named `appc.js` which would live in the root folder of any Appcelerator product's project.

### File Format

The file format would be a standard text file with a `.js` extension to indicate that the source is JavaScript.

### File Processing

The file should be able to be converted from JS into JSON safely.  To support backwards compatiblity and incorporation into products like the Marketplace, we will have a utility to convert an `appc.js` into an `appc.json` which will represent the frozen static state of the metadata for a snapshot.  For example, when the appc cli publishes a component to the marketplace, the marketplace server will store the contents of the JSON in the DB.  With the new format, the conversion utility will convert the appc.js file into the same JSON format (or a modified format if required) for safe processing.

For processing, the static values returned from loading the `appc.js` as a module (the exports object) will be snapshotted.

### Format Limits

Since the file format is JS, any valid JS should be useable in the file for runtime processing.

### Required Properties

The following are required properties at the top-level exported object:

- `type` - the type of project. this will be a set of pre-defined types.
- `group` - the group of product. this will be a set of pre-defined groups.

The following are optional properties at the top-level exported object:

- `dependencies` - node.js package.js style format which describes the AppC component dependencies (if any).
 
Each product will define it's own named top-level key such as `hyperloop` for Hyperloop specific configuration or `titanium` for Titanium specific configuration.

The content inside each product's configuration will be product dependent. Each product will be free to have any mandatory and optional configuration underneath the product's property key.

#### Types

The following are the initial set of pre-defined types:

- `app` - application (such as Titanium)
- `api` - api (such as Arrow)
- `analytics` - analytics (such as Insights)

#### Groups

The following are the initial set of pre-defined groups:

- `titanium`
- `arrow`

The "group" indicates the product family grouping for the project.

### Example

The following is an example of a Hyperloop enabled appc.js:

```javascript
/**
 * Hyperloop configuration
 */
module.exports = {
	type: 'app',
	group: 'titanium',
	dependencies: {
	},
	hyperloop: {
		ios: {
			xcodebuild: {
				/**
				 * any flags available to be passed into the Xcode can be
				 * included here to further customize the xcode build
				 */
				flags: {
					GCC_PREPROCESSOR_DEFINITIONS: 'foo=bar'
				},
				/**
				 * this sample doesn't use StoreKit but this demonstrates
				 * how you can bring in frameworks explicitly. Hyperloop
				 * will automatically determine the required frameworks
				 * but in case you want to force a specific version, you can
				 * include it here
				 */
				frameworks: [
					'StoreKit'
				]
			},
			/**
			 * optionally, you can bring in third-party or first-party libraries,
			 * source code, resources etc. by including them here. The 'key' is the
			 * name of the package that will be used in the require (if code).
			 * the values can either be an Array or String value to the directory
			 * where the files are located
			 */
			thirdparty: {
				'MyFramework': {
					// these can be an array or string
					source: ['src'],
					header: 'src',
					resource: 'src'
				}
			}
		}
	}
};
```

### Justification for JS vs JSON

Why JS vs. JSON?  After experience with various formats (XML, JSON, YAML), I believe it's a good tradeoff to be able to well-document the configuration as well as provide dynamic runtime configuration.  Also, with JS, we can rather easily export the JS and turn the JS into JSON safely for portablity and transportability of the configuration.  This provides a nice tradeoff of the various issues.

### Tooling Support

As part of this proposal, we will need to build a node library to make it easy to "edit" the file by use with tools to preserve comments and the dynamic nature.

The tool would simply use an JS AST parser to build the parse tree and provide a simple mechanism to edit existing values or add new values.  This should be implemented as a node library which can be invoked programatically as an API or via the command line as a lightweight CLI.

## TODO

The following items need to be resolved as part of this CSPEC:

- add a prototype for "editing" a complex JS file programatically
- flesh out a good example of moving `tiapp.xml` to `appc.js`

## Timeline

The goal of this proposal is to incorporate this file format into AppC CLI and Arrow in Q1 2016 and Titanium and Alloy if possible in 6.0 (Q1 2016).

## Status

12-02-2016 - Initial Draft

## Legal Stuff

This proposal is non-binding and may not be implemented, may be implemented partially, differently or not at all. Any intellectual property developed as part of this proposal is owned and Copyright &copy; 2015 by Appcelerator, Inc. All Rights Reserved.

For more information about what a CSPEC is, please visit the [Community Specifications & Proposals Repo](https://github.com/appcelerator/cspec).
