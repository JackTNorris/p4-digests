# p4-digests
Barebones repo for getting started sending and receiving [digest](https://p4.org/p4-spec/p4runtime/main/P4Runtime-Spec.html#sec-digest) messages on a p4 bmv2 software switch. Please run this in the VM provided by P4 in the [p4-tutorials](https://github.com/p4lang/tutorials) repo. Alternatively, you can configure your Ubuntu system with the appropriate tools by following [this tutorial](https://github.com/jafingerhut/p4-guide/blob/master/bin/README-install-troubleshooting.md) if you would prefer to run things on a non-virtual system

# Folder Structure
- src/ => contains relevant p4 source code
    - boilerplate/ => barebone templates for getting started with digest messages
    - pmu_example/ => relatively cleaned up code used for the lab's SGSMA 2024 paper
    - pmu_logging/ => rough code used for Eva's PDC work

# Good to Know Info:
- "pmu_logging" is somewhat defunct in this repo. Please refer to Eva Casto's fork for more information on that functionality
- To adjust the topology of the network adjust the `topology.json` folder in each example's `pod-topo` folder
- By default, the BMV2 implements a 1000ms "buffer" for digest messages. Thankfully, this time can be adjusted. See my post [here](https://forum.p4.org/t/digest-message-propogation-time-from-data-plane-to-runtime-api/927). In short, you'll need to into the code for bmv2 and *rebuild it after changing the "1000ms" flag [here](https://github.com/p4lang/behavioral-model/blob/main/src/bm_sim/learning.cpp#L56-L58)*
## Step by step bmv2 configuration:
- Clone the bmv2 repository onto your local system
- Find [this](https://github.com/p4lang/behavioral-model/blob/main/src/bm_sim/learning.cpp#L56-L58) segment of code, as well as any other segments in the repo that employ a 1000ms delay
- Change the delay to something more reasonable for you purposes
- Follow the instructions in the [README](https://github.com/p4lang/behavioral-model/blob/main/README.md) to rebuild bmv2 (see "3. Building the Code")
- After the `sudo make install` command, navigate to the `/targets/simple_switch` folder in the behavioral-model repo. Update the system binaries with your new compiled binary by running `sudo mv simple_switch /usr/bin/simple_switch`
- Ensure your changes have worked. To do this, check the location of `simple_switch` with the command `whereis simple_switch`, then run `ls -l <the output dir of the command above>` and ensure the listed date is recent (normally, this command is simply `ls -l /usr/bin/simple_switch`, but things might vary depending on your configuration)

# Generic Getting Started Steps:
- Navigate to an item in the src folder (boilerplate, pmu_example, e.t.c)
- Install all necessary pip dependencies using `pip3 install -r requirements.txt`
- Adjust the format of the digest message in the .p4 file (see `digest_message` struct in boilerplate/digest.p4)
- Set logic for sending a digest message (see `send_digest_message` action in boilerplate/digest.p4)
- Parse said digest message in the controller.py file (see `on_digest_recv(msg)` in boilerplate/controller.py)
- Call `make run`
- Get a terminal for each of the devices with the `xterm` command (ex: `xterm h1 h2 s1`)
- Dont forget to load table rules using `simple_switch_CLI < rules.cmd` on `s1`
- Run the controller.py script on s1 (in the s1 terminal, run `python3 controller.py`)
- Navigate to h1 and send a packet to h2 (if you're stuck on how to do this, identify the ip of each mininet host with the ifconfig command)

# PMU_EXAMPLE Instructions & Background:
## How it Works:
A simple 2-host, single switch topology is deployed via mininet (this is the same as the topology used in the basic_forwarding exercise provided by P4)
## How to Run:
- Navigate to the pmu_example folder
- Install pip packages using `pip install -r requirements.txt` (you might need to install pandas and nnpy)
- Call `make run`
- In mininet, call `xterm h1 h2 s1`
- Navigate to s1's terminal
- Run `simple_switch_CLI < rules.cmd` to install the forwarding rules on the device
- Run `python3 controller.py --terminate_after 100` to enable a script that listens for digest messages. The 100 is arbitrary in this case (the program will end after generating 100 packets)
- ^^ If you get python package issues, run `pip3 install` in the xterminal
- Navigate to `h2`'s terminal
- Run `python3 receive.py --terminate_after 30`
- ^^ If you get python package issues, run `pip3 install` in the xterminal
- Navigate to `h1`'s terminal
- Run `python3 send.py pmu12.csv --num_packets 30`
- ^^ If you get python package issues, run `pip3 install` in the xterminal
- Notice that `h2`s terminal has received all packets
- Go into pmu_example/missing-data.json and add in a set of indexes between 1-100
- Re-run the receive.py script
- Re-run the send.py script
- Notice that s1 generate packets to send to h2
- Effectively, if you have 5 missing indexes in your missing indexes file, you only send 95 packets from send.py. Receive.py receives 100 packets, meaning the switch made up the difference

# Allowances and Future Works
Most of this code was hastily written, so *please forgive some of the messiness and redundancies*. I learned a lot about P4 and networking through the process of experimenting with this, so if there's something that isn't making any sense to you, *don't hesitate to make a change -- I likely just forgot to change something*

A few future improvements:
1. Reduce redundant code with common modules shared between all python scripts (was too lazy to figure out nice looking imports, lol)
2. Create a very "thin" superset of P4 which will enable you to import a JSON file model for digest messages in both a P4 file and python controller file
3. Like Andy Fingerhut mentioned, it would be a good idea to look into mirroring packets to the CPU port instead.

# Acknowledgements