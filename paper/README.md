### Previous README

To recreate Fig 2:

- Download and unzip preprocessed Norman2019 dataloader for GEARS [here](https://dataverse.harvard.edu/api/access/datafile/6894431)
- Move the uncompressed `norman2019.tar.gz` folder to `./data`. It should contain the subdirectory `data_pyg`
- Move `essential_norman.pkl` and `go_essential_norman.csv` to `./data`
- So `./data` should contain `essential_norman.pkl`, `go_essential_norman.csv` and `norman2019/data_pyg/`
- Run `fig2_train.py` to train the model

The code here will not install GEARS from the [main repository](https://github.com/snap-stanford/GEARS). It will use the local path to GEARS in this repository `../gears`

For other baselines:
- CPA: See `CPA_reproduce`
- GRN: See `GRN`
- CellOracle: See `CellOracle`

Please raise an issue or email yroohani@stanford.edu in case of any problems/questions

### Our repro attempts

Some of this is also documented in the GEARS github page, issue #5. Followed the instructions above

#### Intended first step is fig2_train.py

I tried running the fig2b adamson and dixit notebooks, but they fail with errors like "ValueError: Could not find project pert_gnn_simulation_adamson2016". I noticed the comment saying "Project is specified by <entity/project-name> please replace it with your project name here". I replaced with my own project name, but then there was no logged data. I concluded that the intended first step is to train the models via fig2_train.py, not to run the notebooks. All good so far -- I wanted to train the models up myself anyway, not just load evaluation results.

#### Environment

I tried running `python3 fig2_train.py` but I needed to install the right set of packages. I used the `dependencies.yaml` file via `mamba env create --file  dependencies.yaml`.

- Note: I have no GPU; this will install torch for CPU. For GPU, see torch official install docs.
- Note: this does not install GEARS itself. I assume fig2_train.py is meant to pull in the copy of gears in this repo via `sys.path.append("../")`.

#### Weaselling out of the GPU requirement

I then ran `python3 fig2_train.py` and `python3 fig2_train.py --device cpu`, but it seems GPU is currently a hard requirement. Wanting a GPU is reasonable but I checked to see how far I could weasel out of it. I modified lines 8 and 47 of fig2_train.py to allow users to pass in 'cpu' as the device, and I modified six lines in utils.py to avoid += as per #9. I also removed many instances of `.to(device)` in `gears/gears.py`, `gears/inference.py`, and `gears/dataloader.py`. This seemed to work -- I hit an issue, but not seemingly GPU-related.

#### Getting the GO graph

The next issue was two hardcoded paths in utils:

    df_jaccard = pd.read_csv('/dfs/user/kexinh/gears2/go_essential_gi.csv')
    df_jaccard = pd.read_csv('/dfs/user/kexinh/gears2/go_essential_all.csv')

I changed these to `data/go_essential_norman.csv` since I have that file from following directions in this README. I notice those may not be actually used, depending on args passed to `get_similarity_network`.

#### Running on adamson and dixit

The default option trains on the Norman data, whereas I am primarily interested in figure 2b showing results on the Adamson and Dixit perturb-seqs. I modified `fig2_train.py` to accept 'adamson' or 'dixit' as input. This likely broke the ability to run on other datasets but I'm ok with that. I run it like this.

    cd paper
    conda activate gears_reproduction
    mkdir logs
    python fig2_train.py --dataset=adamson > logs/adamson.out 2>  logs/adamson.err &
    python fig2_train.py --dataset=dixit >  logs/dixit.out 2>  logs/dixit.err &

I was able to train successfully on Adamson and Dixit, but got stuck reading the results into the Fig2b notebooks.