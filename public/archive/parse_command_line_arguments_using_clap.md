---
title: "Parse command line arguments using clap"
category: 'Rust'
pubDate: 'Mar 30 2024 06:30'
---

For _pomosh_, I wanted to add a command line functionality to be able to do something like the following:

```shell
$ pomosh --version
$ pomosh --help
$ pomosh --disable-chime # starts the session without the alarm
```

I looked up command line arguments in Rust and found `std::env` module in Rust which allows you to process arguments through the command line.

So I used `env` to parse arguments and ran necessary codes for each argument:
```rust
use std::env;

fn main() {
    for arg in args.iter().skip(1) {
        match arg.to_lowercase().as_str() {
            "--version" | "-v" => {
                const VERSION: Option<&str> = option_env!("CARGO_PKG_VERSION");
                println!("{}", VERSION.unwrap_or("unknown"));
                return;
            }
            "--help" | "--" => {
                help();
                return;
            }
            "--chime=false" => {
                config.enable_chime = false;
            }
            "--preset=1" => {
                run_preset = true;
            }
            "--preset=2" => {
                config.focus_duration = 50;
                config.break_duration = 10;
                config.long_break_duration = 20;
                run_preset = true;
            }
            _ => {
                println!(
                    "{}: unexpected argument '{}' found\n",
                    "error".red(),
                    arg.yellow()
                );
                return;
            }
        }
    }
}
```

I wasn't sure whether this approach was the best option. So I asked on elk.

Lots of people told me not to do it this way. Instead, they recommended me to use this crate called [clap](https://docs.rs/clap/latest/clap/index.html) with _derive_ features.

Couple others preferred something different:
- [structopt](https://crates.io/crates/structopt)
- [lexopt](https://docs.rs/lexopt/latest/lexopt/)

I decided to use the most favorite one from the current community, _clap_.

```rust
#[derive(Parser)]
#[command(version, about = "Command line Pomodoro Timer", long_about = None)]
struct Args {
    #[arg(short, long, help="Preset pomodoro (focus/break/long break)\n 1) 25/5/10\n 2) 50/10/20", value_name = "1|2" )]
    preset: Option<u32>,

    #[arg(short, long, help="Disable session/break complete chime", default_value_t = false)]
    mute: bool,
}
```

The above code will generate the following with `--help`:

```textfile
Command line Pomodoro Timer

Usage: pomosh [OPTIONS]

Options:
  -p, --preset <1|2>  Preset pomodoro (focus/break/long break)
                       1) 25/5/10
                       2) 50/10/20
  -m, --mute          Disable session/break complete chime
  -h, --help          Print help
  -V, --version       Print version
```

Much better :)

Now instead of parsing arguments through `env`, I can use the `Args` structure that I created above to check if any other options are enabled from the command line.

Note that I didn't add any codes for `--help` and `--version`.

```rust
fn main() {
    let cli = Args::parse();
    let mut config = SessionConfig::new();
    config.mute = cli.mute;

    if let Some(v) = cli.preset {
        match v {
            1 => {
                run_session(&config);
            },
            2 => {
                config.focus_duration = 50;
                config.break_duration = 10;
                config.long_break_duration = 20;
                run_session(&config);
            },
            _ => {
                eprintln!("{}: '{}' is not a valid preset code.\n\nFor more information, try 'pomosh --help'", "error".red(), v);
            }
        }
    } else {
        config.prompt_for_settings();
        run_session(&config);
    }
}

```

---

Another option.

Texts after the triple slashes (`///`) will be shown in the `--help` manual.

```rust
/// Command line Time Tracker
#[derive(Debug, Parser)]
#[command(version, about, long_about = None)]
struct Cli {
    #[command(subcommand)]
    command: Option<Commands>,
}

#[derive(Subcommand, Debug)]
enum Commands {
    /// Add a task to the tracker
    Add {
        #[arg(short, long, value_name = "TASK" )]
        task: Option<String>,
    },
    /// Deletes a task from the tracker
    Delete {
        #[arg(short, long, value_name = "TASK" )]
        task: Option<String>,
    },
    /// Starts the tracker for <TASK>
    Start {
        #[arg(short, long, value_name = "TASK" )]
        task: Option<String>,
    },
    /// Stops currently running tracker
    Stop {
        #[arg(short, long)]
        task: bool,
    },
    /// List accumulated time for the task or all (default="all")
    Log {
        #[arg(short, long)]
        task: Option<String>,
    }
}

// ...


let cli = Cli::parse();
match &cli.command {
    Some(Commands::Add { task }) => {
        match task {
            Some(t) => shigan.add_task(&(t.to_lowercase())),
            None => println!("None")
        }
    }
    Some(Commands::Delete { task }) => {
        match task {
            Some(t) => shigan.delete_task(&(t.to_lowercase())),
            None => println!("None")
        }
    }
    Some(Commands::Start { task }) => {
        match task {
            Some(t) => shigan.start_task(t.to_lowercase()),
            None => println!("None")
        }
    }
    Some(Commands::Log { task }) => {
        match task {
            Some(t) => shigan.log(&(t.to_lowercase())),
            None => shigan.log(&"all".to_string().to_lowercase())
        }
    }
    Some(Commands::Stop { task: _ }) => {
        shigan.end_task();
    }
    None => {}
}
```