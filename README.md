# Fluence Character Counter

A character count application using Fluence.

[Quick Start](https://doc.fluence.dev/docs/quick-start)

> 
[Aqua Book](https://doc.fluence.dev/aqua-book/)
> Fluence is an open protocol and a framework for internet or private cloud applications. Fluence provides a peer-to-peer development stack so that you can create applications free of proprietary cloud platforms, centralized APIs, and untrustworthy third-parties.
> At the core of Fluence is the open-source language Aqua that allows for the programming of peer-to-peer scenarios separately from the computations on peers.

### Instructions from install
```bash
Source code to learn from:  
    fluence/examples         –  AIR + Rust examples                        
    fluence/aqua-playground  –  Aqua + Typescript examples                
    fluence/fluentpad        -  Frontend + backend project written in Aqua  

To download example projects, run ./.devcontainer/.aux/download_examples.sh  

Available tools:  
    $ marine           – build wasm from Rust  
    $ marine repl      – run wasm services locally  
    $ aqua-cli         – compile Aqua to AIR + Typescript  
    $ fldist           – deploy & query services  
```

### Deploying the HelloWorld example
```bash
# list available network peers
fldist env

# Then we can deploy to a selected peer id
fldist --node-id 12D3KooWSD5PToNiLQwKDXsu8JSysCwUt8BVUJEqCHcDe7P5h45e \
       new_service \
       --ms artifacts/hello_world.wasm:configs/hello_world_cfg.json \
       --name hello-world

# This returns a service id to interact with
service id: 51c4c9a4-2c0f-432f-a7a2-6293c4b57ee9
service created successfully
```

### Fluence Developer Hub

The deployed service is listed with others on the [Fluence Developer Hub](https://dash.fluence.dev/)

```
hello-world 51c4c9a4-2c0f-432f-a7a2-6293c4b57ee9 12D3KooW...7P5h45e /ip4/164.90.171.139/tcp/7770
```

### Character Count Extension

The task is to extend the simple hello world example to add a character count to sent messages.

Within the Docker Container, prepare the version control for the examples code.

```bash
cd /workspaces/devcontainer/examples
rm -r ./.git
git init
git remote add origin git@github.com:ben-razor/Fluence-Service.git
git add .
git commit -m "Initial commit"
git branch -M main
git push -u origin main
```

### Extending The Service

The service that will be deployed resides in the quickstart/3-browser-to-service folder. We will need to modify the code in src/main.rs to add a character count to the message reply.

```rust
#[marine]
pub struct CharCount {
    pub msg: String,
    pub reply: String,
}

#[marine]
pub fn char_count(message: String) -> CharCount {
    let num_chars = message.chars().count();
    let _msg;
    let _reply;

    if num_chars < 1 {
        _msg = format!("Your message was empty");
        _reply = format!("Your message has 0 characters");
    } else {
        _msg = format!("Message: {}", message);
        _reply = format!("Your message {} has {} character(s)", message, num_chars);
    }

    CharCount {
        msg: _msg,
        reply: _reply
    }
}
```



> Before building we must refactor all referencies to hello_world, hello and HelloWorld in the configs directory and Cargo.toml file to use the new char_count module, char_count function name and CharCount struct.

Once this is done we build the service. This will create a char_count.wasm file in ./artifacts that will be deployed later.

```bash
cd quickstart
cd 2-hosted-services
./scripts/build.sh
```

We update the tests in main.rs to work with the new service. We enter a special character in the string to ensure that special characters are being counted correctly. 

Using the following command we can test that our service is working as expected before deployment:

```bash
cargo +nightly test --release
```

### Deployment To Fluence

Once the service is built and the tests pass, we are ready to deploy the service to Fluence.

We can gather a list of test peers using the command:

```bash
fldist env
```

We choose the top entry from the list for the deployment and use the node id with the fldist deployment command:

```bash
fldist --node-id 12D3KooWSD5PToNiLQwKDXsu8JSysCwUt8BVUJEqCHcDe7P5h45e \
       new_service \
       --ms artifacts/char_count.wasm:configs/char_count_cfg.json \
       --name char-count-br
```

This returned the service id **9454c078-1b68-4ae1-bb30-b82690d5fec0** which we can use later to interact with the service. (Your deployed service id will be different, or you can use this one for performing front end tests).

### Fluence Developer Hub

You can ensure your service has been deployed by viewing the [Fluence Developer Hub](https://dash.fluence.dev/)

The deployed contract used in this tutorial can be viewed at:
[Character Count Service](https://dash.fluence.dev/blueprint/5c187205fef8d3306dcf6ffd73efd623a8774d3adc698e6244bc11e81d704ba6)

### Updating The Aqua Code 

With the character count contract deployed, we move to the front end code. This is located in the quickstart/3-browser-to-service directory.

Our changes focus on the getting-started.aqua file. [Aqua](https://doc.fluence.dev/docs/knowledge_aquamarine/hll) is a simplified language for defining peer-to-peer applications with Fluence.

Firstly we change the service id and peer id to match those returned when we deployed the contract.

Secondly, we refactor all references related to hello world to char count versions.

The most important change is that we will now need to pass through our message (messageToSend) and need to update the interface to support this:

```TypeScript
func countChars(messageToSend: string, targetPeerId: PeerId, targetRelayPeerId: PeerId) -> string:
    -- execute computation on a Peer in the network
    on charCountNodePeerId:
        CharCount charCountServiceId
        comp <- CharCount.char_count(messageToSend)

    -- send the result to target browser in the background
    co on targetPeerId via targetRelayPeerId:
        res <- CharCountPeer.char_count(messageToSend)

    -- send the result to the initiator
    <- comp.reply
```

We must now run the aqua compiler which processes the Aqua code and generates a typescript file in src/_aqua/getting-started.ts that we can use in our web app.

```bash
aqua --input ./aqua/getting-started.aqua --output ./src/_aqua/
```

### Updating The Front End Code

Finally we need to update the front end code to interact with our new character count service via the interfaces defined in the Aqua file.

The front end code is found in the src/App.tsx file. For this simplified application this mostly requires changing hello and hello world variable names to their character count alternatives that we updated in the getting-started.aqua file.

We also add a message box so that we can send our custom messages across so that we can now say more than a simple hello.

### Running The Updated Application

The application is started by running:

```
npm start
```

We are presented with a list of relays (you can choose any). Once this is selected we fire up (or get a friend to fire up) another browser window and do the same.

Next we fire off our amazing message to the other peer id:

We get back out message with the characters counted!

And as a bonus our message gets sent through to the other client like magic!

### Wrapping Up

And that's it! I hope this tutorial will help you to make some amazing applications using fluence.
