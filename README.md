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
