# Artifact of paper "Automated Ambiguity Detection in Layout-Sensitive Grammars"

This is the artifact for the paper "Automated Ambiguity Detection in Layout-Sensitive Grammars" at OOPSLA'23. The main purpose of this artifact is to support our evaluation results in §7 (mostly Table 1) and the theoretical results in §3 -- §5 (the main conclusions are Theorem 5.9 and Theorem 5.10).

This artifact consists of two parts (each is a directory):

1. `tool/`: our prototype tool that implements the ambiguity detection approach (following §5), together with necessary data and scripts for reproducing the evaluation (§7);
2. `proof/`: our Coq mechanization (§6) of all the definitions and theorems mentioned in §3 -- §5.

## 1 Getting Started Guide (10 min)

### 1.1 Building Lamb (3 min)

Change the directory to `tool/`. We provide two options to set up the environment and you should choose **either**:

- (1.1a) via [Nix](https://nixos.org/download.html) (version 2.15.1 is tested; latest recommended)
- (1.1b) via Docker

NOTE: As far as we know, Nix (1.1a) is a better option because:

+ you can easily view the images of parse trees via any document/figure preview applications installed on your machine (instead of a virtual machine typically with text UI only);
+ you can use whatever Coq IDEs to step into the Coq proofs, if necessary.

For Windows users, WSL2 is a possible way to install and run Nix. However, if you find difficulties in installing Nix, Docker (1.1b) is the secondary option where you will only be able to view parse trees in S-expression format.

#### 1.1a Via Nix

If you haven't installed Nix, type the following command to do so in your shell:

```bash
curl --proto '=https' --tlsv1.2 -sSf -L https://install.determinate.systems/nix | sh -s -- install
```

Type `nix --version` to make sure `nix` is installed and can be found in the current PATH. In the `tool/` directory, run the following to build our tool:

```bash
nix build --no-link
```

The building process should take no more than 15 minutes. If encountering an error message requesting to enable the experimental features of Nix, refer to [this guide](https://nixos.wiki/wiki/Flakes#Enable_flakes) to enable flakes and Nix commands.

#### 1.1b Via Docker

If you haven't installed Docker, follow [this link](https://docs.docker.com/desktop/install/linux-install/) (you should select the one that matches your OS) to install Docker Desktop and start it.

In the `tool/` directory, run the following to build a Docker image:

```bash
docker build . -t lamb:0.0.0
```

Then, start a container using that image, so that you will be dropped into the development shell:

```bash
docker run -it lamb:0.0.0
```

Your shell prompt should now end with `[lamb-dev]>`.

### 1.2 Testing the Motivating Example (4 min)

Once you have successfully built Lamb, try running the motivating example. File `tests/motivating-example.bnf` contains the grammar used in Section 2 of the paper. You can check its content first, as the notation is slightly different from those used in the paper (see below for details).

We use the following symbols in the plain-text EBNF files:

- `A|>` for offside on `A`
- `A>>` for offside-align on `A`
- `A~` for single on `A`
- `A || B` for `A` align `B`
- `A -> B` for `A` indent `B`

Then, run our tool on that grammar (assuming that your current directory is `tool`):

```
nix run . -- tests/motivating-example.bnf
```

(if you're already inside the development shell, you can also use `python -m lamb tests/motivating-example.bnf` to run our tool)

The output should look like below

```
...lines of solving process omitted...

***
Ambiguous sentence of length 3 found. It shall be listed below.
***

do
nop
nop

***
Found locally ambiguous variable: "new-var-0". It corresponds to token(s) [1, 3] in the ambiguous sentence.
***

NOTE: indexing for tokens in the sentence starts at 1. Spaces in the sentence are denoted as `␣'.
NEXT STEP: Review all parse trees using the following commands (execute line by line):

show tree new-var-0 0
show tree new-var-0 1

Type help for other available commands. Command completion available with TAB key.

Now entering REPL...

smt-ambig>
```

TODO: explain dissimilar subtrees

This indicates that our tool successfully found an ambiguous sentence:

```
do
nop
nop
```

As mentioned in the paper, we only consider dissimilar subtrees, since this (in combination with the reachability condition) is logically equivalent to derivation ambiguity. Considering the dissimilar subtrees that witnesses local ambiguity, two of them exists for the subword $w^{1\dots3}$, and they both has the variable $\text{new-var-0}$ as their root. $\text{new-var-0}$ is a new variable created in the process of translating EBNF grammar into LS2NF grammar. When presenting the parse trees, this transformation is undone -- you'll only see variables and rules in original EBNF grammar in the presented parse trees.

Then, have a look at the parse trees one by one, by using the suggested commands `show tree new-var-0 0` and `show tree new-var-0 1`.  

Press enter after each command. You should see the parse trees generated by our tool Lamb.

Finally, input `exit` to quit our tool.

### 1.3 Checking the Coq Proof Artifact (3 min)

Change the directory to `proof/`. Likewise, we also provide two options to compile the Coq proof and you should choose **either**:

- (1.3a) via [Nix](https://nixos.org/download.html) (version 2.15.1 is tested; latest recommended)
- (1.3b) via Docker

#### 1.3a Via Nix

```bash
nix build
```

If no error messages show, then the theorems are all machine-checked.

#### 1.3b Via Docker

Build the docker image with the following command:

```bash
docker build . -t proof-artifact:0.0.0
```

If no error messages show and you see the following line (or something similar if you are using an older version):

```
 => => naming to docker.io/library/proof-artifact:0.0.0
```

Then the building succeeds and all theorems are machine-checked.

## 2 Step-by-Step Instructions for Reproducing Evaluations (25 min)

### 2.1 Looking at Dataset (? min)

The following table describes the relationship between test cases in directory `tool/tests/` and Fig 6 in the paper.

| #    | Lang    | Location                                             |
| ---- | ------- | ---------------------------------------------------- |
| #1-x | Python  | `tool/tests/checker-benchmark/python/x.bnf`          |
| #2-x | SASS    | `tool/tests/checker-benchmark/sass/x.bnf`            |
| #3-x | YAML    | `tool/tests/checker-benchmark/yaml/x.bnf`            |
| #4-x | F#      | `tool/tests/checker-benchmark/fsharp-snippet/x.bnf`  |
| #5-x | Haskell | `tool/tests/checker-benchmark/haskell-snippet/x.bnf` |

The EBNF files (with file suffix `.bnf`) are written in plain-text form, which is a little bit different from the notation in the paper. A comparision between the plain-text form and the form used in the paper have been mentioned in Section 1.2 of this README.

In each group, the first EBNF grammar (`0.bnf`) is the original grammar (i.e., *seed* grammar), while the other four grammars are variants of the seed grammar. This is in accordance with the evaluation setup in the paper.

### 2.2 Running the Experiments (? min)

We assume that you are currently at the `tool` directory. First, load the development shell for our tool:

```bash
nix develop
```

This command introduces the fixed version of relevant tools, like Python, Z3 and Graphviz, into the PATH environment variable. It also sets up Python correctly, so it can find all depended PyPI packages. Conceptually, it is analogous to Virtualenv and Anaconda environment, but Nix can ensure reproducibility while both Virtualenv and Anaconda cannot.

You may notice that after executing this command, the prompt line should then end with `[lamb-dev]>`, which means the development shell has been activated.

TODO: explain run_tests.py; without `--all` we do not run test cases like Python and SASS; timeout;

you can choose whether `--all` should be added

Then, execute the following command to run all but Python / SASS testcases.

```bash
python run_tests.py
```

This shall take around 10-25 minutes, depending on the performance of your platform. Afterwards, the total running time (sum of `solve_time` and `other_time`), ambiguous sentence length, among other details, shall be printed to the terminal. Another copy will also be stored into `result.json` in the current directory. 

To run every testcases including Python and SASS, please run the following command.

```bash
python run_tests.py --all
```

Note that this can take very long (>24h) to complete.

TODO:
1. Set up a timeout in our experiment script, so that we hopefully to reproduce everything within a few hours, not a day.
2. If the reviewer does not want to run all, provide a script to execute a set of quick examples -- maybe the two fastest examples from each lang? -- so that everything is done in one hour.

### 2.3 Checking the Results (? min)

(convert into CSV!)

The results of step 2.2 are saved into a JSON file `result.json`. It has the exact same structure with Table 1 in our paper.

Note: Despite the randomness of the found satisfiable model by Z3 solver, the lengths of the ambiguous sentence (i.e., the last column of Table 1) are determined, though under the hood the found ambiguous sentence may differ.

TODO: we should have a script that does the above, which saves reviewer's time to check data.

### 2.4 Human Understanding of Ambiguous Sentence (? min)

This section presents how one can analyze the cause of ambiguity by inspecting the parse trees for the generated ambiguous sentence, as mentioned in section 7.2. This is a minor result of our evaluation: skip it if your time is tight.

TODO: details

### 2.5 Paper-to-Coq Correspondence Guide (30 min)

Change the directory to `proof/`. Open `README.md` and read the chapter [Paper-to-Artifact Correspondence Guide](proof/README.md#paper-to-artifact-correspondence-guide). If you wish to step into the proofs, launch your Coq IDE in the Nix development shell (by `nix develop`; see [Step Into the Code](proof/README.md#step-into-the-code) for more details).