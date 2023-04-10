# Assemblyline incident manager
This repository contains three Python scripts used for bulk triaging file using Assemblyline.
1. Submitter (`al-incident-submitter`): pushes files from a directory to an Assemblyline instance for analysis.
2. Result analyzer (`al-incident-analyzer`): pulls the submissions from the Assemblyline instance and reports on if the submissions are safe/unsafe.
3. Downloader (`al-incident-downloader`): downloads files submitted to Assemblyline that are under a certain score threshold, matching the folder structure of the files as they were submitted.

# Prerequisites
- You will need the URL of an Assemblyline instance that you have an account on, for best results make sure it is loaded with your best YARA rules, Sandboxes etc. 
  - Want to create your own Assemblyline instance? [HOW-TO](https://cybercentrecanada.github.io/assemblyline4_docs/installation/deployment/)
- You will need two API keys generated by Assemblyline, ideally one with read access and another with write access. 
  - The Write-only key will be used for the "Submitter" and the Read-only key will be used for the "Result Analysis" and the "Downloader".
  - This helps in the context of incident response to reduce the exposure of your Assemblyline instance.

# Installation
## Linux
- Install the following packages: `libffi-dev`, `libssl-dev`
  - (APT) `sudo apt-get install libffi-dev libssl-dev python3`
  - (YUM) `sudo yum install libffi-dev libssl-dev python3`
- Upgrade PIP: `python3 -m pip install --upgrade pip`
- `python3 -m pip install assemblyline-incident-manager`

## Windows
- Download and install the most recent Python .msi installer from https://www.python.org/downloads/release. 
- Upgrade PIP: `python -m pip install --upgrade pip`
- `python -m pip install assemblyline-incident-manager`

# Usage
## Submitter
```
al-incident-submitter --help
Usage: al-incident-submitter [OPTIONS] COMMAND [ARGS]...

  Example 1: al-incident-submitter --url="https://<domain-of-Assemblyline-instance>"  --username="<user-name>" --apikey="/path/to/file/containing/apikey" --classification="<classification>" --service_selection="<service-name>,<service-name>" --path="/path/to/scan" --incident_num=123

  Example 2: al-incident-submitter --config al_config.toml --path="/path/to/scan" --incident_num=123

Options:
  -c, --config FILE         Read option defaults from the specified TOML file
                            [default: ~/al_incident_config.toml]
  --url TEXT                The target URL that hosts Assemblyline.
                            [required]
  --username TEXT           Your Assemblyline account username.  [required]
  --apikey TEXT             A path to a file that contains only your
                            Assemblyline account API key. NOTE that this API
                            key requires write access.  [required]
  --ttl INTEGER             The amount of time that you want your Assemblyline
                            submissions to live on the Assemblyline system (in
                            days).
  --classification TEXT     The classification level for each file submitted
                            to Assemblyline.  [required]
  --service_selection TEXT  A comma-separated list (no spaces!) of service
                            names (case-sensitive) to send files to. If not
                            provided, all services will be selected.
  -t, --is_test             A flag that indicates that you're running a test.
  --path PATH               The directory path containing files that you want
                            to submit to Assemblyline.  [required]
  -f, --fresh               Restart ingestion from the beginning.
  --incident_num TEXT       The incident number for each file to be associated
                            with.  [required]
  --resubmit-dynamic        All files that score higher than 500 will be
                            resubmitted for dynamic analysis.
  --alert                   Generate alerts for this submission.
  --threads INTEGER         Number of threads that will ingest files to
                            Assemblyline.
  --dedup_hashes            Only submit files with unique hashes. If you want
                            100% file coverage in a given path, do not use
                            this flag
  --priority INTEGER        Provide a priority number which will cause the
                            ingestion to go to a specific priority queue.
  --do_not_verify_ssl       Ignore SSL errors (insecure!)
  --help                    Show this message and exit.
```

## Analyzer
```
al-incident-analyzer --help
Usage: al-incident-analyzer [OPTIONS] COMMAND [ARGS]...

  Example 1: al-incident-analyzer --url="https://<domain-of-Assemblyline-instance>" --username="<user-name>" --apikey="/path/to/file/containing/apikey" --incident_num=123

  Example 2: al-incident-analyzer --config ~/al_config.toml --incident_num=123 --min_score=100

Options:
  -c, --config FILE    Read option defaults from the specified TOML file
                       [default: ~/al_incident_config.toml]
  --url TEXT           The target URL that hosts Assemblyline.  [required]
  -u, --username TEXT  Your Assemblyline account username.  [required]
  --apikey PATH        A path to a file that contains only your Assemblyline
                       account API key. NOTE that this API key requires write
                       access.  [required]
  --min_score INTEGER  The minimum score for files that we want to query from
                       Assemblyline.
  --incident_num TEXT  The incident number that each file is associated with.
                       [required]
  -t, --is_test        A flag that indicates that you're running a test.
  --do_not_verify_ssl  Ignore SSL errors (insecure!)
  --help               Show this message and exit.
```

Now check the `report.csv` file that was created. This file will contain what files are safe/unsafe.

## Downloader
```
al-incident-downloader --help
Usage: al-incident-downloader [OPTIONS] COMMAND [ARGS]...

  Example 1: 
  al-incident-downloader --url="https://<domain-of-Assemblyline-instance>" --username="<user-name>" --apikey="/path/to/file/containing/apikey" --incident_num=123 --max_score=100 --download_path=/path/to/where/you/want/downloads --upload_path=/path/from/where/files/were/uploaded/from

  Example 2: 
  al-incident-downloader --config al_config.toml --incident_num=123 --max_score=100 --download_path=/path/to/where/you/want/downloads --upload_path=/path/from/where/files/were/uploaded/from

Options:
  -c, --config FILE             Read option defaults from the specified TOML
                                file  [default: ~/al_incident_config.toml]
  --url TEXT                    The target URL that hosts Assemblyline.
                                [required]
  -u, --username TEXT           Your Assemblyline account username.
                                [required]
  --apikey PATH                 A path to a file that contains only your
                                Assemblyline account API key. NOTE that this
                                API key requires read access.  [required]
  --max_score INTEGER           The maximum score for files that we want to
                                download from Assemblyline.  [required]
  --incident_num TEXT           The incident number that each file is
                                associated with.  [required]
  --download_path PATH          The path to the folder that we will download
                                files to.  [required]
  --upload_path PATH            The base path from which the files were
                                ingested from on the compromised system.
                                [required]
  -t, --is_test                 A flag that indicates that you're running a
                                test.
  --num_of_downloaders INTEGER  The number of threads that will be created to
                                facilitate downloading the files.
  --do_not_verify_ssl           Ignore SSL errors (insecure!)
  --help                        Show this message and exit.
```

If you check the download path you supplied, you should have all files downloaded there.

----------------------------

# L'assistant à la réponse aux incidents d'Assemblyline
Ce répertoire contient trois scripts Python pour assisté le triage de grande quantité de fichiers avec Assemblyline.
1. Soumission (`al-incident-submitter`): envoi les fichiers contenu dans un dossier vers une instance Assemblyline pour l'analyze.
2. Résultats d'analyse (`al-incident-analyzer`): analyse les soumissions et génère un rapport.
3. Téléchargeur (`al-incident-downloader`): télécharge les fichiers sous un certain pointage en préservant la structure original.

# Prérequis
- Vous aurez besoin d'un instance d'Assemblyline à jour et avec vos meilleurs règles YARA, "Sandboxes" etc.
  - Voici comment crée vôtre propre instance: [LIEN](https://cybercentrecanada.github.io/assemblyline4_docs_fr/docs/installation.html)
- Nous vous recommandons d'utilisé deux clé d'api, un `write only` et une `read only`
  - La clé `Write-only` sera utilisé pour soumettre vos fichier avec le script "Submitter" et la clé `Read-only` sera pour "Result Analysis" et le "Downloader".
  - Cette séparation aidera a securisé vôtre instance Assemblyline dans un context de réponse aux incidents

# Installation
## Linux
- Installé les packages suivants: `libffi-dev`, `libssl-dev`
    - (APT) `sudo apt-get install libffi-dev libssl-dev python3`
    - (YUM) `sudo yum install libffi-dev libssl-dev python3`
- Mise à jour de PIP: `python3 -m pip install --upgrade pip`
- `python3 -m pip install assemblyline-incident-manager`

## Windows
- Installé Python 3: https://www.python.org/downloads/release. 
- Mise à jour de PIP: `python -m pip install --upgrade pip`
- `python -m pip install assemblyline-incident-manager`

# Utilisation
## Submitter
```
al-incident-submitter --help
Usage: al-incident-submitter [OPTIONS] COMMAND [ARGS]...

  Example 1: al-incident-submitter --url="https://<domain-of-Assemblyline-instance>"  --username="<user-name>" --apikey="/path/to/file/containing/apikey" --classification="<classification>" --service_selection="<service-name>,<service-name>" --path="/path/to/scan" --incident_num=123

  Example 2: al-incident-submitter --config al_config.toml --path="/path/to/scan" --incident_num=123

Options:
  -c, --config FILE         Read option defaults from the specified TOML file
                            [default: ~/al_incident_config.toml]
  --url TEXT                The target URL that hosts Assemblyline.
                            [required]
  --username TEXT           Your Assemblyline account username.  [required]
  --apikey TEXT             A path to a file that contains only your
                            Assemblyline account API key. NOTE that this API
                            key requires write access.  [required]
  --ttl INTEGER             The amount of time that you want your Assemblyline
                            submissions to live on the Assemblyline system (in
                            days).
  --classification TEXT     The classification level for each file submitted
                            to Assemblyline.  [required]
  --service_selection TEXT  A comma-separated list (no spaces!) of service
                            names (case-sensitive) to send files to. If not
                            provided, all services will be selected.
  -t, --is_test             A flag that indicates that you're running a test.
  --path PATH               The directory path containing files that you want
                            to submit to Assemblyline.  [required]
  -f, --fresh               Restart ingestion from the beginning.
  --incident_num TEXT       The incident number for each file to be associated
                            with.  [required]
  --resubmit-dynamic        All files that score higher than 500 will be
                            resubmitted for dynamic analysis.
  --alert                   Generate alerts for this submission.
  --threads INTEGER         Number of threads that will ingest files to
                            Assemblyline.
  --dedup_hashes            Only submit files with unique hashes. If you want
                            100% file coverage in a given path, do not use
                            this flag
  --priority INTEGER        Provide a priority number which will cause the
                            ingestion to go to a specific priority queue.
  --do_not_verify_ssl       Ignore SSL errors (insecure!)
  --help                    Show this message and exit.
```

## Analyzer
```
al-incident-analyzer --help
Usage: al-incident-analyzer [OPTIONS] COMMAND [ARGS]...

  Example 1: al-incident-analyzer --url="https://<domain-of-Assemblyline-instance>" --username="<user-name>" --apikey="/path/to/file/containing/apikey" --incident_num=123

  Example 2: al-incident-analyzer --config ~/al_config.toml --incident_num=123 --min_score=100

Options:
  -c, --config FILE    Read option defaults from the specified TOML file
                       [default: ~/al_incident_config.toml]
  --url TEXT           The target URL that hosts Assemblyline.  [required]
  -u, --username TEXT  Your Assemblyline account username.  [required]
  --apikey PATH        A path to a file that contains only your Assemblyline
                       account API key. NOTE that this API key requires write
                       access.  [required]
  --min_score INTEGER  The minimum score for files that we want to query from
                       Assemblyline.
  --incident_num TEXT  The incident number that each file is associated with.
                       [required]
  -t, --is_test        A flag that indicates that you're running a test.
  --do_not_verify_ssl  Ignore SSL errors (insecure!)
  --help               Show this message and exit.
```

Regardez le rapport dans `report.csv`. Ce fichier contient un rapport des détections.

## Downloader
```
al-incident-downloader --help
Usage: al-incident-downloader [OPTIONS] COMMAND [ARGS]...

  Example 1: 
  al-incident-downloader --url="https://<domain-of-Assemblyline-instance>" --username="<user-name>" --apikey="/path/to/file/containing/apikey" --incident_num=123 --max_score=100 --download_path=/path/to/where/you/want/downloads --upload_path=/path/from/where/files/were/uploaded/from

  Example 2: 
  al-incident-downloader --config al_config.toml --incident_num=123 --max_score=100 --download_path=/path/to/where/you/want/downloads --upload_path=/path/from/where/files/were/uploaded/from

Options:
  -c, --config FILE             Read option defaults from the specified TOML
                                file  [default: ~/al_incident_config.toml]
  --url TEXT                    The target URL that hosts Assemblyline.
                                [required]
  -u, --username TEXT           Your Assemblyline account username.
                                [required]
  --apikey PATH                 A path to a file that contains only your
                                Assemblyline account API key. NOTE that this
                                API key requires read access.  [required]
  --max_score INTEGER           The maximum score for files that we want to
                                download from Assemblyline.  [required]
  --incident_num TEXT           The incident number that each file is
                                associated with.  [required]
  --download_path PATH          The path to the folder that we will download
                                files to.  [required]
  --upload_path PATH            The base path from which the files were
                                ingested from on the compromised system.
                                [required]
  -t, --is_test                 A flag that indicates that you're running a
                                test.
  --num_of_downloaders INTEGER  The number of threads that will be created to
                                facilitate downloading the files.
  --do_not_verify_ssl           Ignore SSL errors (insecure!)
  --help                        Show this message and exit.
```

Tous les fichiers sans détections seront téléchargé dans le dossier choisi.

