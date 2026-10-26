# Data source

This folder contains a small participant subset from the OpenNeuro dataset
[ds004504](https://openneuro.org/datasets/ds004504/versions/1.0.9):

> Miltiadous, A., Tzimourta, K. D., Afrantou, T., et al. (2023).
> *A Dataset of Scalp EEG Recordings of Alzheimer's Disease, Frontotemporal
> Dementia and Healthy Subjects from Routine EEG*. Data, 8(6), 95.
> [doi:10.3390/data8060095](https://doi.org/10.3390/data8060095).
> OpenNeuro DOI: [10.18112/openneuro.ds004504.v1.0.9](https://doi.org/10.18112/openneuro.ds004504.v1.0.9).

The dataset is released under CC0. Please cite the dataset and associated
publication when using it.

## Included sample

Three participants were randomly selected from each group for this sample:

| Group code | Group | Participants |
| --- | --- | --- |
| A | Alzheimer's disease | sub-004, sub-012, sub-020 |
| C | Healthy controls | sub-037, sub-040, sub-055 |
| F | Frontotemporal dementia | sub-066, sub-068, sub-088 |

Each `sub-*/eeg/` folder contains the EEGLAB `.set` recording and its BIDS
JSON metadata and channel TSV. These are continuous, eyes-closed resting EEG
recordings sampled at 500 Hz with 19 EEG channels. Only these nine participants'
EEG files are included; the full participant table and other recordings are
not copied here.

The recordings are raw dataset files. The dataset README describes a separate
preprocessed set in `derivatives/`; it is not included in this sample.

## How these files were obtained

The dataset was cloned from the OpenNeuro GitHub mirror at tag `1.0.9`.
Participant groups were read from `participants.tsv`; the nine IDs above were
selected at random once and are fixed so the sample can be reproduced. Only
their `.set` recordings were retrieved from the public `s3-PUBLIC` Git-annex
remote. Each recording and its `_eeg.json` and `_channels.tsv` sidecars were
copied into this folder, preserving the BIDS-style `sub-*/eeg/` structure.
No other participants' EEG recordings or the full participant table were
copied into this project.

## Re-create this sample (Windows PowerShell)

Run these commands from the root of this project. They require Git and `uv`.
If Git-annex is not installed, install it once with:

```powershell
uv tool install git-annex
```

Clone the exact dataset release to a new temporary folder:

```powershell
$dataset = Join-Path $env:TEMP ("ds004504-v1.0.9-" + [guid]::NewGuid().ToString("N"))
git clone --branch 1.0.9 https://github.com/OpenNeuroDatasets/ds004504.git $dataset
```

Enable the public annex remote and retrieve only the nine selected signal
files (not the recordings for the other participants):

```powershell
git -C $dataset annex enableremote s3-PUBLIC

$participants = @(
    "sub-004", "sub-012", "sub-020",
    "sub-037", "sub-040", "sub-055",
    "sub-066", "sub-068", "sub-088"
)
$signalFiles = foreach ($participant in $participants) {
    "$participant/eeg/${participant}_task-eyesclosed_eeg.set"
}
git -C $dataset annex get --from=s3-PUBLIC -- $signalFiles
```

Copy each signal and its metadata sidecars into this project's `data/`
directory:

```powershell
$projectData = Join-Path (Get-Location).Path "data"
foreach ($participant in $participants) {
    $source = Join-Path $dataset "$participant\eeg"
    $destination = Join-Path $projectData "$participant\eeg"
    New-Item -ItemType Directory -Path $destination -Force | Out-Null

    $baseName = "${participant}_task-eyesclosed"
    foreach ($suffix in @("_eeg.set", "_eeg.json", "_channels.tsv")) {
        $fileName = "$baseName$suffix"
        Copy-Item -LiteralPath (Join-Path $source $fileName) `
            -Destination (Join-Path $destination $fileName) -Force
    }
}
```

The clone contains dataset metadata and annex pointers for all participants;
the `git annex get` command above downloads signal content only for the nine
listed IDs. The `.json` and `.tsv` sidecars are copied from the clone and are
not annexed signal recordings.

## Re-create this sample (Bash/Linux)

Run these commands from the root of this project. They require Git, Bash, and
Git-annex. Install Git-annex using your Linux distribution's package manager,
or with `uv` if it is available:

```bash
uv tool install git-annex
```

Clone the exact dataset release to a temporary folder:

```bash
dataset="$(mktemp -d "${TMPDIR:-/tmp}/ds004504-v1.0.9.XXXXXX")"
git clone --branch 1.0.9 \
  https://github.com/OpenNeuroDatasets/ds004504.git "$dataset"
```

Enable the public annex remote and retrieve only the nine selected signal
files (not the recordings for the other participants):

```bash
git -C "$dataset" annex enableremote s3-PUBLIC

participants=(
  sub-004 sub-012 sub-020
  sub-037 sub-040 sub-055
  sub-066 sub-068 sub-088
)
signal_files=()
for participant in "${participants[@]}"; do
  signal_files+=("$participant/eeg/${participant}_task-eyesclosed_eeg.set")
done
git -C "$dataset" annex get --from=s3-PUBLIC -- "${signal_files[@]}"
```

Copy each signal and its metadata sidecars into this project's `data/`
directory:

```bash
project_data="$PWD/data"
for participant in "${participants[@]}"; do
  source="$dataset/$participant/eeg"
  destination="$project_data/$participant/eeg"
  mkdir -p "$destination"

  base_name="${participant}_task-eyesclosed"
  for suffix in _eeg.set _eeg.json _channels.tsv; do
    cp "$source/$base_name$suffix" "$destination/"
  done
done
```

The clone contains dataset metadata and annex pointers for all participants;
the `git annex get` command above downloads signal content only for the nine
listed IDs. The `.json` and `.tsv` sidecars are copied from the clone and are
not annexed signal recordings.
