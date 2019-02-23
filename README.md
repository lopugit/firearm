# firearm
Wrapper around [GunDB](https://github.com/amark/gun) to provide alternative syntax, promise support, utility modules, and easy Gun adapter events.

# Rewriting Gun docs: #

## Original: ##

    var gun = Gun();

    gun.get('mark').put({
      name: "Mark",
      email: "mark@gunDB.io",
    });

    gun.get('mark').on(function(data, key){
      console.log("update:", data);
    });

## With Firearm: ##

    var firearm = new Firearm()

    firearm.mark = {
      name: "Mark",
      email: "mark@gunDB.io",
    }

    firearm.mark.on(data => {
      console.log("update:", data);
    })

## Second Example: ##

    var cat = {name: "Fluffy", species: "kitty"};
    var mark = {boss: cat};
    cat.slave = mark;

    // partial updates merge with existing data!
    gun.get('mark').put(mark);

    // access the data as if it is a document.
    gun.get('mark').get('boss').get('name').once(function(data, key){
      // `val` grabs the data once, no subscriptions.
      console.log("Mark's boss is", data);
    });

    // traverse a graph of circular references!
    gun.get('mark').get('boss').get('slave').once(function(data, key){
      console.log("Mark is the slave!", data);
    });

    // add both of them to a table!
    gun.get('list').set(gun.get('mark').get('boss'));
    gun.get('list').set(gun.get('mark'));

    // grab each item once from the table, continuously:
    gun.get('list').map().once(function(data, key){
      console.log("Item:", data);
    });

    // live update the table!
    gun.get('list').set({type: "cucumber", goal: "scare cat"});

## With Firearm: ##

    var cat = { name: 'Fluffy', species: 'kitty' }
    var mark = { boss: cat }
    cat.slave = mark

    // partial updates merge with existing data!
    firearm.mark = mark

    // access the data as if it is a document.
    let marksBoss = await firearm.mark.boss.name.value
    console.log("Mark's boss is", marksBoss)

    // add both of them to a table!
    firearm.list.set(firearm.mark.boss)
    firearm.list.set(firearm.mark)

    // grab each item once from the table, continuously:
    firearm.list.map().once(data => {
      console.log("Item:", data)
    })

    // live update the table!
    firearm.list.set({ type: "cucumber", goal: "scare cat" })

# How is this possible? #
It's simple really. Firearm wraps the Gun instance with a Proxy. Each property lookup (using dot notation or bracket) calls a `gun.get()` and returns the Proxy to be used for chaining. Any call to `firearm.value` will return a promise getter of `gun.once()`.

# Utilities: #
- `.value` - Promise getter for `.once()`; `let cats = await firearm.cats.value`
- `.remove()` - `firearm.cats.remove()`

# Writing your own Gun adapter, made easy: #
One of the best features of Firearm is to write Gun adapters without having to know too much about the Gun constructor, proper placement to initialize your adapter, and bugs caused by not forwarding the events. Firearm takes care of all of this for you! Simply define a `Function` or `Class`, return an object that contains the events you want to hook into, and the rest is up to you.

Example class:

    class gunAdapter {
      constructor(firearm, opts, context) {
        return {
          events : {
            // Storage
            get: function(...),
            put: function(...),
            
            // Wire
            in: function(...),
            out: function(...),
          }
        }
      }
    }

Example function:

    function gunAdapter(firearm, opts, context) {
      return {
        events: { 
          // Storage
          get: function(...),
          put: function(...),
            
          // Wire
          in: function(...),
          out: function(...),
        }
      }
    }

To use your adapter, include it in your project then:<br>
`firearm.extend(gunAdapter)`

# More coming soon! #
Firearm expects to be a wrapper for other utility functions, offering an easy way to extend either firearm or gun via Proxies or directly. This will allow you to create custom firearm methods for wrapping verbose syntaxes.

Some utility proposals have already emerged, such as handling arrays in a more suitable fashion.

    firearm.cats.sparky = { color: 'orange' }
    firearm.cats.howie = { color: 'white' }

    firearm.mark.cats = [firearm.cats.sparky, firearm.cats.howie]

Another utility proposal was RPC

    // local
    firearm.rpc.host('peerName')
    firearm.rpc.register('procName', function(data) {})

    // Remote
    firearm.rpc.exec('procName', { some: 'data' })
    // or
    firearm.rpc.select('peerName').exec('procName')

<br>

## License: ##
[MIT](https://github.com/lopugit/firearm/blob/master/LICENSE)
