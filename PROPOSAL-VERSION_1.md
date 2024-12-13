# Transit: Plugin Architecture Version 1
Transit plugin architecture is meant to provide either a shared or seperate (a shared getter must be present either way) environment. In this environment, plugins will put their own data (eg. Functions, Arbitrary data, etc.) and do it's own managing.

### Ruleset:
1. Transit system must provide a shared read-only environment. (eg. `Transit.shared`)
   > This is needed for plugins to see and interact with other plugins, preventing certain errors and race conditions.
2. Transit environment must provide a controller in "any" type. (eg. `Transit.env().actionExtensions`)
   > This is needed to give support for plugins to have plugins.
3. Transit system must provide a method to create an environment or to serve a shared environment.
   > This is needed since it's the core functionality.
4. Transit plugin must provide an ID
5. Transit plugin must provide a init method (eg. `plugin ( environment: (ByRef)TransitEnvironment )`)

### Example:
Here's an example written in Javascript that provides a Transit System.
```js
class Transit {
    constructor () {
        this.envList = {}
    }
    env () {
        return {
            actions: {}
        };
    }
    addTransit ( data ) {
        if ( !data?.id )
            throw new Error( 'Transit data does not contain an ID property.' );
        if ( this.envList[ data?.id ] )
            throw new Error( `Transit with the ID "${ data.id }" already exists in the transit pool.` );
        if ( !data?.plugin || typeof data.plugin !== 'function' )
            throw new Error( 'Transit data has an invalid plugin method.' );
        const env = this.env();
        data.plugin( env );
        return this.envList[ data.id ] = env;
    }
    get shared () {
        const obj = {
            actions: {}
        };
        for ( const [ id, transit ] of Object.entries( this.envList ) ) {
            for ( const [ actionId, actionInfo ] of Object.entries( transit.actions || {} ) ) {
                if ( obj.actions[ actionId ] )
                    throw new Error( `Action with the ID "${ actionID }" already exists in the transit pool.` );
                obj.actions[ actionId ] = actionInfo;
            }
        }
        return obj;
    }
}
```
In this example, we will assume that the Transit system is initialized as `transit`.
Here's an example Transit plugin that is compatible with the example above:
```js
// in test.transit.js
module.exports = {
    id: 'MyPlugin',
    plugin ( env ) {
        env.actions.log = ( ...args ) => console.log( ...args );
    }
}
```
Now that we have our plugin, let's put it into test
```js
const plugin = require( './test.transit.js' );
transit.addTransit( plugin );

transit.shared.actions.log( "hello", "world" );
// console output: hello world
```
