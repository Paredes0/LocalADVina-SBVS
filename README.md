# Local SBVS Pipeline 🚀

A complete workflow for **Structure-Based Virtual Screening (SBVS)** that combines the convenience of Google Colab with the power of your local computer (If you have it ;)).

This project was born to solve, on one hand, the long compute times, interruptions, and limits of free cloud platforms, and on the other, the inconvenience of usability. Using a hybrid architecture, this pipeline allows for efficient, robust, and resumable execution of massive docking analyses.

The code was developed using an AI-assisted programming approach (*vibe coding*) with advanced language models, resulting in a functional notebook optimized for research.

## Why did I create this project?

Before explaining exactly what this is and how it works, I thought about contextualizing this project, and I think the best way to do so is to briefly explain who I am and why I created this.

I am a final-year biotechnology student, currently doing an extracurricular internship in a group dedicated to structural bioinformatics and computational chemistry called [BIO-HPC](https://bio-hpc.eu/), at the [UCAM](https://www.ucam.edu/) university. During these internships, I encountered difficulties when using free online SBVS tools. Therefore, with the help of artificial intelligence (Gemini 2.5 pro) and applying my basic programming knowledge, I created this notebook in Google Colab.

The fact that it is designed to connect to your personal computer is due to the low computing capacity of the free version of Google Colab. At the end of this README.md, I indicate which CPU I used for these calculations.

I hope this is useful to someone as it has been to me.

## ✨ Key Features

* **Molecule Preparation**: Converts receptors and ligands (from PDB and SDF) to `PDBQT` format using **Meeko**.
* **Single Docking**: Performs a test docking with **AutoDock-Vina** to validate the configuration.
* **Library Processing**: Efficiently prepares libraries of thousands of ligands for screening.
* **High-Throughput SBVS**: Runs massive virtual screening with **Smina**, a Vina fork optimized for speed.
* **Result Refinement**: Selects the best candidates and performs a more exhaustive docking for high-quality predictions.
* **Parallelized Calculation**: Leverages all local CPU cores thanks to Python's `multiprocessing` module.
* **Resumable System**: Includes functions to resume interrupted calculations, avoiding progress loss.

## ⚙️ Architecture and Setup

The system uses Google Colab as the user interface and a local machine for heavy computing. To replicate the environment, follow these steps:

### 1. Local Environment Setup (WSL)

A Linux environment is required. It is recommended to use the **Windows Subsystem for Linux (WSL)**.

### 2. Package Manager

Install **Micromamba** for faster and more efficient package management. It is absolutely necessary to create environments and execute commands safely and efficiently.

Once WSL is installed, configure and open the terminal to install the necessary environments to work:

```bash
# Micromamba installation example
"${SHELL}" <(curl -L micro.mamba.pm/install.sh)

# Update micromamba to the latest version
micromamba self-update

# Give execution permissions to Micromamba
chmod +x bin/micromamba

```

### 3. Create the Virtual Environment

Create a virtual environment with all the necessary bioinformatics tools.

> **Important:** the notebook cells run their commands inside an environment
> named **`docking_vina`**. Use exactly this name, or the cells will not find
> your tools.

#### Option A — One command (recommended)

A pinned [`environment.yml`](environment.yml) is provided so you can build the
whole toolchain in a single step:

```bash
# Create the environment from the file (it is named docking_vina)
micromamba create -f environment.yml

# Activate the environment
micromamba activate docking_vina
```

#### Option B — Manual installation

```bash
# Create the environment
micromamba create -n docking_vina

# Activate the environment
micromamba activate docking_vina

# Install packages from the conda-forge and bioconda channels, plus pip
micromamba install -c conda-forge -c bioconda numpy pandas swig boost-cpp libboost tqdm rdkit meeko openbabel smina
pip install vina

```

> **Why these packages?** `rdkit`, `meeko` and `openbabel` prepare and convert
> molecules; `vina` and `smina` run the docking; `pandas` and `tqdm` are
> imported by the screening scripts the notebook generates. Installing them all
> now avoids errors part-way through a long run.

### 4. Link Colab with the Local Environment

To connect the Colab interface with the power of your machine, you need to start a Jupyter server.

```bash
# If you are still in the docking_vina environment, you must exit it
micromamba deactivate docking_vina

# Create the environment
micromamba create -n colab_connect

# Activate the environment
micromamba activate colab_connect

# Install JupyterLab in your environment and htop to monitor processes
micromamba install jupyterlab htop

# Start the Jupyter server
jupyter lab --no-browser --NotebookApp.allow_origin='https://colab.research.google.com' --port=8888

```

Follow the instructions in the terminal to obtain the URL with the token and connect from Google Colab (`Connect to a local runtime`).

(Simply copy and paste the link appearing in the terminal that contains "http://localhost:8888/lab?token..." into the space available for a URL in Google Colab within the (`Connect to a local runtime`) menu).

### 5. Regular Usage

The first thing you must do before anything else is download the `.ipynb` file and upload it to Google Colab via --> "File -> Upload notebook" and select the file.

Once you have both environments installed (`docking_vina` and `colab_connect`) with all packages installed, you will only have to perform the previous step without needing to install JupyterLab again; just activate the `colab_connect` environment, start the Jupyter server, and paste it into Google Colab to connect.

```bash
# Activate the environment
micromamba activate colab_connect

# Start the Jupyter server securely
jupyter lab --no-browser --NotebookApp.allow_origin='https://colab.research.google.com' --port=8888

```

The `docking_vina` environment will work by executing orders sent from the Google Colab notebook to the JupyterLab server. You do not need to connect to it from the terminal; just leave the `colab_connect` environment active and the server started.

### 6. File Organization and Paths

Perhaps the trickiest part at the beginning is organizing all the paths to folders and files to perform jobs within the GC notebook itself. For some cells, examples are given of what the path should look like (e.g., in "executable path", which I had to include due to execution issues), but not in others.

For the notebook to work correctly, it is crucial to understand how WSL accesses your Windows files. The path to your `C:` drive will be `/mnt/c/` and to your `D:` drive will be `/mnt/d/`.

It is recommended to create a main folder for the project. For example, on your `D:` drive:

D:/Bioinformatics/My_SBVS_Project/

* data/
* receptor.pdb
* ligands.sdf


* results/
* docking_results.csv
* best_ligands/



When the notebook asks for a path, you must use the WSL format. For example, the path to `receptor.pdb` would be:
`/mnt/d/Bioinformatics/My_SBVS_Project/data/receptor.pdb`

## 🚀 Workflow in the Notebook

Once you have connected your local environment, the process inside the Google Colab notebook is as follows:

1. **Cell 1: Path Configuration**: The first step is to edit the variables containing the paths to your files (receptor, ligands, executables, and result folders). Make sure to use the WSL path format (`/mnt/c/...`).
2. **Cell 2: Molecule Preparation**: Run this cell to convert your receptor and ligands to `PDBQT` format.
3. **Cell 3: Test Docking**: Performs a docking with a single ligand to ensure the `bounding box` and configuration are correct.
4. **Cell 4: Massive Virtual Screening (SBVS)**: This cell will start the heavy calculation on your local computer. Remember that this process can take hours depending on the number of ligands and parameters used (exhaustiveness and conformations).
5. **Cell 5: Analysis and Refinement**: Once the screening is finished, this section allows you to analyze the results, select the best candidates, and perform a higher-precision docking with AutoDock Vina.

## ⚠️ WARNING: How to Stop a Local Execution

When you start a script from Google Colab, it runs on your PC independently. **Stopping the cell in Colab will NOT stop the process on your computer.** To stop a long execution (like a virtual screening), you must do it manually in your WSL terminal.

You have two main ways:

### Option 1: The Drastic Option (Close Jupyter Server) 🚨

This is the fastest way to stop everything at once.

1. Go to the terminal where you have the `jupyter lab` server running.
2. Press `Ctrl + C`.
3. It will ask if you want to stop the server. Type `y` and press `Enter`.

* **Consequence:** This will kill the Jupyter server and usually all processes it started. You will lose the connection to Colab and will have to restart the server to run anything again.

### Option 2: The Precise and Recommended Option (Use `htop`) ✅

This method allows you to stop a specific script without taking down the Jupyter server, giving you more control.

1. **Open a NEW WSL terminal.** (Leave the Jupyter terminal running untouched).
2. **Start the process monitor installed previously alongside JupyterLab:**
```bash
htop

```


3. **Find your script:** Inside `htop`, you have two ways to find the process:
* **Filter (F4):** Press `F4` and type the name of your script, for example: `run_sbvs.py`.
* **Sort (F6):** Press `F6` and select `PERCENT_CPU` to sort by CPU usage. Your script should appear at the top.


4. **Stop the process:**
* Use the arrows to select the main process (e.g., `python run_sbvs.py`).
* Press `F9` (Kill).
* Make sure signal `15 SIGTERM` (orderly termination) is selected and press `Enter`.



With this method, you only stop the calculation you are interested in and keep the Jupyter server ready to receive new orders.

## 💻 Performance

Performance tests show notable efficiency:

* **~30' per 1000 ligands**.
* **Test Hardware**: Intel(R) Core(TM) i5-7400 CPU @ 3.00 GHz (4 cores / 4 threads).
* **It is recommended to have between 8 and 16 GB of RAM**.

## 🛠️ Tech Stack

| Tool | Purpose |
| --- | --- |
| **AutoDock-Vina** | Precision molecular docking |
| **Smina** | High-performance docking for SBVS |
| **Meeko** | Molecule preparation (PDBQT) |
| **RDKit** | Cheminformatics and library management |
| **WSL** | Linux environment on Windows |
| **Conda/Micromamba** | Package and environment management |
| **JupyterLab** | Server for remote connection |
| **Google Colab** | User interface and control |

## 🤝 Contributing

Contributions, bug reports, and ideas are welcome! Please read
[`CONTRIBUTING.md`](CONTRIBUTING.md) for how to set up the environment and open
issues or pull requests. There are issue templates for bug reports and feature
requests to help you get started.

## 📚 How to Cite

If this pipeline is useful in your research, please cite it (see
[`CITATION.cff`](CITATION.cff)) **and** the underlying scientific tools listed
in the [Acknowledgements](#acknowledgements-licenses-and-responsibilities)
section below.

## 📄 License

Distributed under the MIT License. See the `LICENSE` file for more information.

## 👨‍💻 Author

**Noé Paredes Alfonso**

* **GitHub**: [Paredes0](https://github.com/Paredes0)
* **LinkedIn**: [Noé Paredes Alfonso](https://www.linkedin.com/in/no%C3%A9-paredes-alfonso-395328267/)

## Acknowledgements, Licenses, and Responsibilities

This project is a computational pipeline that integrates multiple open-source and scientific software tools. Its operation depends entirely on the exceptional work of the developers and communities maintaining these packages. Below are the key dependencies, their licenses, and the correct way to cite their work.

### Main Scientific Tools

This workflow uses the following programs for structure preparation, molecular docking, and virtual screening. It is strongly recommended to cite the original publications if you use the results of this pipeline in your research.

* **Smina**
* **Function:** Used for high-throughput virtual screening and refinement re-docking.
* **License:** [Apache License 2.0](https://github.com/smina/smina/blob/master/LICENSE)
* **Academic Citation:** Koes, D. R., Baumgartner, M. P., & Camacho, C. J. (2013). Lessons learned in empirical scoring with smina from the CSAR 2011 benchmarking exercise. *Journal of chemical information and modeling*, 53(8), 1893-1904.


* **AutoDock Vina**
* **Function:** Used for single-molecule docking.
* **License:** [Apache License 2.0](https://github.com/ccsb-scripps/AutoDock-Vina?tab=Apache-2.0-1-ov-file#)
* **Academic Citation:** Trott, O., & Olson, A. J. (2010). AutoDock Vina: improving the speed and accuracy of docking with a new scoring function, efficient optimization, and multithreading. *Journal of computational chemistry*, 31(2), 455-461.


* **Meeko**
* **Function:** Preparation of receptor and ligand molecules to convert them to PDBQT format.
* **License:** [GNU General Public License v3.0](https://github.com/forlilab/Meeko/blob/master/LICENSE)


* **RDKit: Open-Source Cheminformatics**
* **Function:** Used for molecular structure handling (SDF file reading) and as a fundamental dependency of Meeko.
* **License:** [BSD 3-Clause License](https://github.com/rdkit/rdkit/blob/master/license.txt)


* **Open Babel**
* **Function:** Used for format conversion, specifically to generate SMILES from PDBQT results.
* **License:** [GNU General Public License v2.0](http://openbabel.org/wiki/License)
* **Academic Citation:** O'Boyle, N. M., et al. (2011). Open Babel: An open chemical toolbox. *Journal of Cheminformatics*, 3(1), 33.



### Environment and Support Libraries

Execution and data analysis are possible thanks to the following tools:

* **Mamba / Micromamba**
* **Function:** Package and environment management, allowing robust installation of complex scientific dependencies.
* **License:** [BSD 3-Clause License](https://github.com/mamba-org/mamba/blob/main/LICENSE)


* **Pandas**
* **Function:** Used for data manipulation, analysis, and result export.
* **License:** [BSD 3-Clause License](https://github.com/pandas-dev/pandas/blob/main/LICENSE)


* **NumPy**
* **Function:** Fundamental dependency of Pandas for numerical calculation.
* **License:** [BSD 3-Clause License](https://github.com/numpy/numpy/blob/main/LICENSE.txt)


* **Tqdm**
* **Function:** Provides progress bars to monitor long processes.
* **License:** [MIT License](https://github.com/tqdm/tqdm/blob/master/LICENCE)



### Disclaimer

This software is provided "AS IS", without warranty of any kind, express or implied. The results generated by this pipeline (such as binding affinities) are theoretical predictions and require experimental validation to be confirmed.
