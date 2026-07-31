## Architecture walk

fragarach is a software that has basically 2 purposes: 

1. It can run a target binary and check if its malicious by detecting the pattern of syscalls it makes
2. It can take a binary along with a flag that indicates whether its benign or malware and use the information to record its patterns to train its neural network

