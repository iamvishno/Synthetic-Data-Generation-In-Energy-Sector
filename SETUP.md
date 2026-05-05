# SETUP — Synthetic Energy Data project



## 0. One-time prerequisites

You only need to do these once on this machine. Skip anything you already have.

| Tool               | Required version                            | Verify with                   |
| ------------------ | ------------------------------------------- | ----------------------------- |
| Python             | 3.10 (any patch)                            | `py -3.10 --version`          |
| NVIDIA driver      | 555 or newer                                | `nvidia-smi`                  |
| VS Code            | latest                                      | open it                       |
| VS Code extensions | Python, Jupyter, Pylance (all by Microsoft) | install from Extensions panel |

If `py -3.10 --version` errors with "Requested Python version (3.10) not installed",
install Python 3.10 from <https://www.python.org/downloads/release/python-31011/>
(tick "Add python.exe to PATH" during installation).

---

## 1. Open the project in VS Code and open a terminal

1. *File → Open Folder…* and select `<PROJECT FOLDER>`.
2. Open a PowerShell terminal: `` Ctrl+` ``.
3. The terminal prompt should end in `<PROJECT FOLDER>`.

Run this sanity block:

```powershell
py -3.10 --version
nvidia-smi
```

You should see `Python 3.10.x` and your RTX 5060 listed in the NVIDIA table
with a CUDA Version of 12.8 or higher in the top-right of the table.

---

## 2. Allow PowerShell to run activation scripts (one-time)

```powershell
Set-ExecutionPolicy -Scope CurrentUser RemoteSigned
```

If it asks for confirmation, type `Y` and press Enter.



## 4. Create the project virtual environment with Python 3.10

This creates a fresh, properly isolated environment named `synthetic-energy-venv`.

```powershell
py -3.10 -m venv synthetic-energy-venv
```

When done, the folder `synthetic-energy-venv\` should appear in the project root.

---

## 5. Activate the environment

```powershell
.\synthetic-energy-venv\Scripts\Activate.ps1
```

Your prompt should now begin with `(synthetic-energy-venv)`.

> **Always activate before working.** Every new VS Code terminal needs you to
> run this one line again. Once `(synthetic-energy-venv)` is in the prompt,
> you are inside the project environment.

---

## 6. Confirm the environment is properly isolated (critical)

This is the step that broke last time. Verify all three lines below pass.

```powershell
python -c "import sys; print('python exe :', sys.executable)"
type synthetic-energy-venv\pyvenv.cfg
python -c "import torch" 2>&1
```

**Expected results:**

- The `python exe` path **must** include `synthetic-energy-venv\Scripts\python.exe`.
- The contents of `pyvenv.cfg` should include `include-system-site-packages = false`.
- The third command should error with `ModuleNotFoundError: No module named 'torch'`.
  This is *desired* — it confirms the new venv has no inherited torch from your
  system Python. If it shows a torch version, **stop** and tell Claude.

---

## 7. Upgrade pip inside the venv

```powershell
python -m pip install --upgrade pip
```

---

## 8. Install PyTorch with CUDA 12.8 (Blackwell support for RTX 5060)

This installs the GPU build, not the CPU build. ~2.8 GB download.

```powershell
pip install --no-cache-dir torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu128
```

---

## 9. Verify PyTorch sees the GPU

```powershell
python -c "import torch; print('version :', torch.__version__); print('cuda    :', torch.cuda.is_available()); print('device  :', torch.cuda.get_device_name(0) if torch.cuda.is_available() else 'NONE')"
```

**Expected output:**

```
version : 2.x.x+cu128
cuda    : True
device  : NVIDIA GeForce RTX 5060 ...
```

If `cuda` says `False`, **stop** here and paste the output to Claude.
We must not move on without GPU acceleration confirmed.

---

## 10. Install the rest of the project dependencies

```powershell
pip install -r requirements.txt
```

This pulls numpy, pandas, matplotlib, seaborn, scikit-learn, jupyter,
scipy, statsmodels, POT (Wasserstein distance), tqdm, holidays, and friends.

---

## 11. Register the venv as a Jupyter kernel

Run this once. It makes the kernel selectable from the VS Code notebook UI.

```powershell
python -m ipykernel install --user --name synthetic-energy --display-name "Python (synthetic-energy)"
```

---

## 12. Final verification — one command, must pass

Copy the entire block (PowerShell handles the multi-line string) and paste:

```powershell
python -c @"
import sys, importlib, platform
print('=' * 60)
print('Python  :', platform.python_version(), 'at', sys.executable)
print('=' * 60)
pkgs = ['numpy','pandas','scipy','matplotlib','seaborn','sklearn','torch','tqdm','statsmodels','holidays','ot']
for p in pkgs:
    try:
        m = importlib.import_module(p)
        print(f'  OK   {p:<14}  {getattr(m, \"__version__\", \"?\")}')
    except Exception as e:
        print(f'  FAIL {p:<14}  {e}')
import torch
print('-' * 60)
print('Torch  :', torch.__version__)
print('CUDA OK:', torch.cuda.is_available())
if torch.cuda.is_available():
    print('GPU    :', torch.cuda.get_device_name(0))
    cap = torch.cuda.get_device_capability(0)
    mem = torch.cuda.get_device_properties(0).total_memory / (1024**3)
    print(f'Compute: sm_{cap[0]}{cap[1]}')
    print(f'VRAM   : {mem:.1f} GiB')
    x = torch.randn(2048, 2048, device='cuda'); y = x @ x.T; torch.cuda.synchronize()
    print('Matmul :', tuple(y.shape), 'OK on GPU')
print('=' * 60)
print('READY' if torch.cuda.is_available() else 'NOT READY (CUDA missing)')
"@
```

**Expected last line:** `READY`



## Re-activating in future sessions

Any time you reopen the project in a new terminal:

```powershell
.\synthetic-energy-venv\Scripts\Activate.ps1
```

That's the only line you ever need to repeat.

---

## Troubleshooting cheat sheet

| Symptom                                                                     | Most likely cause                | Fix                                                                              |
| --------------------------------------------------------------------------- | -------------------------------- | -------------------------------------------------------------------------------- |
| `cuda: False` even after step 9                                             | Wrong PyTorch wheel installed    | Repeat step 8 with `--no-cache-dir`, then step 9                                 |
| Activation script "not recognized"                                          | Wrong path or wrong venv name    | Make sure you're in `MY PROJECT` and the folder `synthetic-energy-venv` exists   |
| `Requested Python version (3.10) not installed`                             | Python 3.10 missing              | Install from python.org and tick "Add to PATH"                                   |
| `ModuleNotFoundError: No module named 'torch'` inside the notebook          | Wrong kernel selected            | Click kernel picker top-right of notebook → choose **Python (synthetic-energy)** |
| Pip warns `Ignoring invalid distribution -ip`                               | Corrupted leftover from old venv | Delete the venv folder and redo from step 3                                      |
| `no kernel image is available for execution on the device` at training time | Wrong CUDA build of torch        | Verify section 9 output really says `+cu128`; if not, redo step 8                |
