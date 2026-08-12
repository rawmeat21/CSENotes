## 1. Committing During Network Interruptions or Traveling

**Scenario:** You are working on a critical fix during a flight, in a cafe with unstable Wi-Fi, or during an ISP outage.

- **CVCS Problem:** You make 15 logical modifications over 6 hours. Because `svn commit` requires a live connection to the central server, you **cannot commit locally**.
    
- **Impact:**
    
    - You are forced to keep all changes in a massive, uncommitted "dirty" working directory.
        
    - You cannot create atomic, incremental commits (e.g., _Refactor API_, _Add Tests_, _Fix Edge Case_).
        
    - If your OS crashes or your drive fails before you connect to the network, your entire work is lost—the system had no record of it.


## 2. Server Infrastructure Outages Block the Entire Team

**Scenario:** The central SVN/Perforce server suffers a database corruption, disk failure, or network interface crash at 9:00 AM.

- **CVCS Problem:** Version control operations across the entire company come to a hard stop.
    
- **Impact:**
    
    - Developers cannot commit code, switch features, view past logs (`svn log`), or check who modified a line (`svn blame`).
        
    - If the central server's backups are corrupted, **the entire history of the project is lost forever**, even if 50 developers have the current code checkout on their laptops.


## 3. High Latency and Slow Local Operations

**Scenario:** You want to run a history search (`svn log`), compare your local file to a commit from two weeks ago (`svn diff -r 4521`), or view a file's history.

- **CVCS Problem:** The client does not store the repository graph locally. Every inspection query must send a network payload to the central server, wait for the server to traverse its database, and stream the response back.
    
- **Impact:**
    
    - Commands that execute in **milliseconds** in Git (reading from local `.git/objects`) take **seconds or minutes** in a CVCS over high-latency or remote connections.
        
    - Tools like `git bisect` (binary searching history to locate bugs) are impractical or extremely slow in a CVCS due to constant network round-trips for every checkout.


## 4. Heavyweight, Risky Branching and Merging

**Scenario:** You want to experiment with a risky refactoring idea without affecting your team, or your team manages multiple parallel release versions.

- **CVCS Problem:** In systems like SVN, a "branch" is traditionally implemented as a copy of a directory tree on the server.
    
- **Impact:**
    
    - **High Overhead:** Creating branches requires network communication and server-side tracking.
        
    - **Painful Merging:** Merge algorithms in older CVCS implementations lack advanced Directed Acyclic Graph (DAG) tracking. Merging a long-lived branch back into the main trunk often results in complex, manual conflict resolution loops (often called "Merge Hell").
        
    - **Behavioral Inhibition:** Because branching and merging are painful, developers avoid branching altogether, leading to long-lived, unstable central trunks where half-finished code is committed just to "save work."


## 5. File Locking Bottlenecks (Exclusive Checkout)

**Scenario:** Two developers need to edit different parts of the same core configuration file or source file simultaneously.

- **CVCS Problem:** To prevent merge conflicts on complex or central files, many CVCS workflows rely on **exclusive file locks** (`svn lock` / Perforce checkout locks).
    
- **Impact:**
    
    - Developer A locks `config.json` to make a change.
        
    - Developer B is completely blocked from modifying `config.json` until Developer A finishes, commits, and releases the lock. If Developer A goes to lunch or forgets to release the lock, Developer B’s workflow stalls.