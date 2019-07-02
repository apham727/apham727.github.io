---
layout: post
title: Virtual Filesystem
description: >
  A filesystem abstraction modeled after unix filesystems
image: /assets/img/vfs.png
noindex: true
---

During the later part of my time at the Rice Efficient Computing Group, I developed the virtual filesystem to expose kernel components in an organized manner and to temporarily store files. Essentially, the virtual filesystem I wrote serves as the layer between the user interface and more concrete filesystems, such as a network filesystem or the physical filesystem on disk. (The link to the github repo for this project can be found [here](https://github.com/apham727/virtual-filesystem){:target="_blank"}).

One of the most difficult (and simultaneously easiest) parts of this project was writing it in Rust. Rust has a super strict typesystem, so it made it difficult unifying the different types of files and directories across one single tree structure, and it necessitated the extensive use of pointers and type-casting. However, Rust's memory safety guarantees allowed me to code pointer-heavy structures without fear of null-pointer errors or other memory-related bugs common to C-languages. 

This project also led to my first [Stack Overflow question](https://stackoverflow.com/questions/53216593/vec-of-generics-of-different-concrete-types). (I ended up sticking with the Vec<Box<Node<Other>>> idea so that more classes could implement the Node trait in the future).



## Notes on Implementation
The VFS project led to the development of certain common unix commands for traversing filesystems, such as *cd*, *ls*, *mkdir*, *rm*, and *pwd*. By far the most difficult command to implement was *cd*, which *c*hanges *d*irectories. It biggest reason for this was because it involved creating the logic for relative/absolute paths. It was a thorough exercise in string parsing, recursion, tree traversal, and error handling. For example, here is the *get* function, which would return the node based on the wd (working directory) parameter:

~~~rust
    /// Gets the reference to the directory specified by the path given the current working directory 
    pub fn get(&self, wd: &StrongAnyDirRef) -> Result<FSNode, &'static str> {
        let current_path;
        { current_path = Path::new(wd.lock().get_path_as_string());}
        
        // Get the shortest path from self to working directory by first finding the canonical path of self then the relative path of that path to the 
        let shortest_path = match self.canonicalize(&current_path).relative(&current_path) {
            Some(dir) => dir, 
            None => {
                error!("cannot canonicalize path {}", current_path.path); 
                return Err("couldn't canonicalize path");
            }
        };

        let mut new_wd = Arc::clone(&wd);
        debug!("components {:?}", shortest_path.components());
        let mut counter: isize = -1;
        for component in shortest_path.components().iter() {
            counter += 1; 
            // Navigate to parent directory
            if component == ".." {
                let dir = match new_wd.lock().get_parent_dir() {
                    Ok(dir) => dir,
                    Err(err) => {
                        error!("failed to move up in path {}", current_path.path);
                        return Err(err)
                        }, 
                };
                new_wd = dir;
            }
            // Ignore if no directory is specified 
            else if component == "" {
                continue;
            }

            // Navigate to child directory
            else {
                // this checks the last item in the components to check if it's a file
                // if no matching file is found, advances to the next match block
                if counter as usize == shortest_path.components().len() - 1  && shortest_path.components()[0] != ".." { // FIX LATER
                    let children = new_wd.lock().list_children(); // fixes this so that it uses list_children so we don't preemptively create a bunch of TaskFile objects
                    for child_name in children.iter() {
                        if child_name == component {
                            match new_wd.lock().get_child(child_name.to_string(), false) {
                                Ok(child) => match child {
                                    FSNode::File(file) => return Ok(FSNode::File(Arc::clone(&file))),
                                    FSNode::Dir(dir) => {
                                        return Ok(FSNode::Dir(Arc::clone(&dir)));
                                    }
                                },
                                Err(err) => return Err(err),
                            };                       
                        }
                    }
                }
                               
                let dir = match new_wd.lock().get_child(component.clone().to_string(),  false) {
                    Ok(child) => match child {
                        FSNode::Dir(dir) => dir,
                        FSNode::File(_file) => return Err("shouldn't be a file here"),
                    }, 
                    Err(err) => return Err(err),
                };
                new_wd = dir;
            }

        }
        return Ok(FSNode::Dir(new_wd));
    }
}
~~~
This function wasn't even a part of the public interface, but it was one of many helper functions that did a lot of the behind-the-scenes work for the virtual filesystem. 

## Takeaway
Over the 4+ month-long development process, I changed the public interface at least 10 times. Every time I changed it, something would break internally and would require hours of debugging to find the error. 

The biggest lesson I learned from this project was to create an interface at the onset that is flexible enough to support future endeavours so that I can avoid days' worth of debugging in the future. 