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
