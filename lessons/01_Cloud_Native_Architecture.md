
# Course intro
Welcome!

## Goals

* Get new people interested in software infrastructure
* Provide an update on where the industry is heading
* Try to piece togheter parts of infastructure to solve problems
* Hopefully, by the end you will have an idea of what DevOps is, it's role and get interested in infastrucutre development.

## Organization

* We will get together each Friday for 2 hours.
* Work will continue between these meetings.
  * We'll try to collaborate using slack.
  * If we get stuck we'll spend the first 15 minutes of each meeting to try and fix them.
* Courses will cover mostly the theory of how things work, what problem do they solve and the solution space.
* All information is in github.com

## Tools

* [ngrok](https://ngrok.com/) -> secure tunnel between your laptop and desktop.
  * https://stackoverflow.com/questions/42442320/ssh-tunnel-to-ngrok-and-initiate-rdp 

# Why DevOps

* Started with this prezentation in 2009: [10+ Deploys Per Day: Dev and Ops Cooperation at Flickr](https://youtu.be/LdOe18KhtT4)
* A set of patterns and practicies to enable development teams to move faster and deliver more inovation [Systems for Innovation - Adrian Cockcroft, at USI](https://youtu.be/-vlOG3UIp9c)
* A dayer look for companies, you either inovate, move quickly and stay in the game or someone else will render you irrelevant.
* A look at the a traditional company : [The Phoenix Project: A Novel about IT, DevOps, and Helping Your Business Win](https://www.amazon.com/Phoenix-Project-DevOps-Helping-Business/dp/0988262592)

# Cloud Native Architecture

## What is a cloud native architecture

### The CNCF perspective:
Cloud native computing uses an open source software stack to be:
* Containerized. Each part (applications, processes, etc) is packaged in its own container. This facilitates reproducibility, transparency, and resource isolation.
* Dynamically orchestrated. Containers are actively scheduled and managed to optimize resource utilization.
* Microservices oriented. Applications are segmented into microservices. This significantly increases the overall agility and maintainability of applications.

### The reality of things

Most of the time you will be working on an platform that is moving toward this.  Also you probably don't need everything so it's important to understand what to select, what works for you and what complicates things rather and provide any benefits. It's a thin line to walk so hopefully at the end you'll understand the problem space and what makes sense to use.

## Walking though the application stack

* A good place to start is the CNCF (Cloud Native Computing Foundation) landscape project: https://github.com/cncf/landscape. These cover the major tooling needs and offer some solutions for each problem space.
