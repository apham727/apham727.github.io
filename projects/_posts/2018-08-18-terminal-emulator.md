---
layout: post
title: Terminal Emulator/CLI
description: >
  A terminal with support for unix-like commands
image: /assets/img/terminal.png
noindex: true
---

The code for the terminal can be found [here](https://github.com/apham727/terminal-emulator).

The terminal was my first major project in Rust as part of the Rice Efficient Computing Group. My objective was to create a unix-like terminal to allow users to run commands/applications in the operating system. 

# Starting Out
I had available three libraries (or "crates" in Rust): 
1. library to capture keypresses
2. library to display text to a display buffer
3. library to run kernel tasks

# Stages of Implementation
I began by creating the functionality to run commands in the kernel. Using the 3rd library, I wrote a function to validate an input against existing commands and then execute the preset command upon validation. 

With this functionality in place, I moved to emulating terminal behavior. This involved recording user input, tracking/resetting prompts, mapping keypresses to certain actions, and echoing keypresses back to the terminal. It resulted in a very rudimentary terminal; however, the code was extremely messy and confusing. As a result, I added in the idea of standard input/output in the terminal for greater orthogonality. 

The final major stage was to beef up the terminal. I implemented a few common unix commands, better error handling, and some other features you would find in an everyday terminal like command history. 

# Notes 
The most difficult part in the early stages of the terminal was working with raw strings. Since at the end of the day, the display buffer only displayed the raw string that was passed to it, I had to meticulously index the display string to prevent out-of-bounds and overlap errors. 

Another concern that arose in the later stages of development was the separation between external libraries and internal components and commands. Take the *ls* command, for example. It would make sense to put *ls* as a separate application in a different crate than the terminal, but that would also incur large overheads everytime the command was invoked (because the kernel has to start a new process and context switch). However, leaving the logic inside the terminal would bloat the code. 
For decisions like this, we would bias towards externalizing a lot of this logic for the sake of maintainability. 

# Takeaways
Real-time input handling is hard. For example, this was the code that redirected keypresses and ran built-in functions. 
~~~rust
    loop {
        // Handle cursor blink
        if let Some(text_display) = terminal.window.get_displayable(&display_name){
            text_display.cursor_blink(&(terminal.window), FONT_COLOR, BACKGROUND_COLOR);
        }

        // Handles events from the print queue. The queue is "empty" is peek() returns None
        // If it is empty, it passes over this conditional
        if let Some(print_event) = terminal.print_consumer.peek() {
            match print_event.deref() {
                &Event::OutputEvent(ref s) => {
                    terminal.push_to_stdout(s.text.clone());

                    // Sets this bool to true so that on the next iteration the TextDisplay will refresh AFTER the 
                    // task_handler() function has cleaned up, which does its own printing to the console
                    terminal.refresh_display(&display_name);
                    terminal.correct_prompt_position = false;
                },
                _ => { },
            }
            print_event.mark_completed();
            // Goes to the next iteration of the loop after processing print event to ensure that printing is handled before keypresses
            continue;
        } 


        // Handles the cleanup of any application task that has finished running, including refreshing the display
        terminal.task_handler()?;
        if !terminal.correct_prompt_position {
            terminal.redisplay_prompt();
            terminal.correct_prompt_position = true;
        }
        
        // Looks at the input queue from the window manager
        // If it has unhandled items, it handles them with the match
        // If it is empty, it proceeds directly to the next loop iteration
        let event = match terminal.window.get_key_event() {
            Some(ev) => {
                ev
            },
            _ => { continue; }
        };

        match event {
            // Returns from the main loop so that the terminal object is dropped
            Event::ExitEvent => {
                trace!("exited terminal");
                window_manager::delete(terminal.window)?;
                return Ok(());
            }

            Event::ResizeEvent(ref _rev) => {
                terminal.refresh_display(&display_name); // application refreshes display after resize event is received
            }

            // Handles ordinary keypresses
            Event::InputEvent(ref input_event) => {
                terminal.handle_key_event(input_event.key_event, &display_name)?;
                if input_event.key_event.action == KeyAction::Pressed {
                    // only refreshes the display on keypresses to improve display performance 
                    terminal.refresh_display(&display_name);
                }
            }
            _ => { }
        }
~~~

I learned to be careful about what functions I was calling in this loop to prevent "keypress lag", because each 
iteration of this loop would track a single keypress. It was a good exercise in writing efficient code! 