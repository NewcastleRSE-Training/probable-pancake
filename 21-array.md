---
title: Parallelising with Job Arrays.
teaching: 15
exercises: 5
---



::::::::::::::::::::::::::::::::::::::: objectives

- What are job arrays?
- What benefit does job arrays bring?
- What type of jobs would benefit from job arrays?

::::::::::::::::::::::::::::::::::::::::::::::::::

:::::::::::::::::::::::::::::::::::::::: questions

- Prepare a job submission script for an array job.

::::::::::::::::::::::::::::::::::::::::::::::::::

```bash
#!/bin/bash
#SBATCH --partition=short_free
#SBATCH --job-name=serial
#SBATCH --nodes=1
#SBATCH --tasks=1
#SBATCH --cpus-per-task=1

# Do a word frequency analysis of the collected works of Shakespeare

DATA_FILE=data.1

echo "Starting word frequency analysis of $DATA_FILE"
echo "=============================================="

time cat $DATA_FILE | \
	sed s'/\ /\n/g' | \
	tr -c -d "[A-Za-z\n]" | \
	tr [A-Z] [a-z] | \
	sort | \
	strings -n 1 | \
	uniq -c | \
	sort -n > data.out

echo "====================================="
echo "Completed word analysis of $DATA_FILE"
```

:::::::::::::::::::::::::::::::::::::::: keypoints

- Stuff

::::::::::::::::::::::::::::::::::::::::::::::::::
