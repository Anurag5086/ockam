```
title: Secure Channel over many hops
```

# Secure Channel over many hops

## App worker

Create a new file at:

```
touch examples/06-secure-channel-many-hops.rs
```

Add the following code to this file:

```rust
// examples/06-secure-channel-many-hops.rs

use ockam::{Context, Result, Route, SecureChannel};
use ockam_get_started::{Echoer, Hop};

#[ockam::node]
async fn main(mut ctx: Context) -> Result<()> {
    // Start an echoer worker.
    ctx.start_worker("echoer", Echoer).await?;

    // Start hop workers - hop1, hop2, hop3.
    ctx.start_worker("hop1", Hop).await?;
    ctx.start_worker("hop2", Hop).await?;
    ctx.start_worker("hop3", Hop).await?;

    SecureChannel::create_listener(&mut ctx, "secure_channel_listener").await?;

    let route_to_listener =
        Route::new()
            .append("hop1")
            .append("hop2")
            .append("hop3")
            .append("secure_channel_listener");
    let channel = SecureChannel::create(&mut ctx, route_to_listener).await?;

    // Send a message to the echoer worker via the channel.
    ctx.send(
        Route::new()
            .append(channel.address())
            .append("echoer"),
        "Hello Ockam!".to_string()
    ).await?;

    // Wait to receive a reply and print it.
    let reply = ctx.receive::<String>().await?;
    println!("App Received: {}", reply); // should print "Hello Ockam!"

    ctx.stop().await
}
```

To run this new node program:

```
cargo run --example 06-secure-channel-many-hops
```

Note the message flow.

## Message Flow

![](./sequence.svg)

<div style="display: none; visibility: hidden;">
<hr><b>Next:</b> <a href="../07-routing-over-transport">07. Routing over a transport</a>
</div>