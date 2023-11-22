# assign-resources

This crate contains a macro to help assign and split up resources from a
struct, such as the `Peripherals` struct provided by embedded PACs and HALs,
into many smaller structs which can be passed to other tasks or functions.

It's best explained with the example below. Here we define new structs named
`UsbResources` and `LedResources`, each containing some IO pins and a
peripheral, and generate a new `split_resources!()` macro. When called,
the `split_resources!()` macro takes `PA12`, `PA11`, and `USB` out of
`p: Peripherals` and uses them to create the field `usb: UsbResources`, and
similarly creates the field `leds: LedResources` in the returned object. We can
then move these new structs into our tasks, which access the resources by name.

```rust
use embassy_stm32::peripherals;

assign_resources! {
    usb: UsbResources {
        dp: PA12,
        dm: PA11,
        usb: USB,
    }
    leds: LedResources {
        r: PA2,
        g: PA3,
        b: PA4,
        tim2: TIM2,
    }
}

#[embassy_executor::task]
async fn usb_task(r: UsbResources) {
    // use r.dp, r.dm, r.usb
}

#[embassy_executor::task]
async fn led_task(r: LedResources) {
    // use r.r, r.g, r.b, r.tim2
}

#[embassy_executor::main]
async fn main(spawner: embassy_executor::Spawner) {
    let p = embassy_stm32::init(Default::default());
    let r = split_resources!(p);
    spawner.spawn(usb_task(r.usb)).unwrap();
    spawner.spawn(led_task(r.leds)).unwrap();

    // can still use p.PA0, p.PA1, etc
}
```

This has a few advantages: you only need to write the specific pin names like
`PA12` in one place and can refer to them by name thereafter, you only have one
argument for each task instead of potentially very many, and you don't need
to write out lots of code to split the resources up. If you're targetting
multiple different hardware, you can use `#[cfg]` to change pin allocations
in just one place.
