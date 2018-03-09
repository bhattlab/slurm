# slurm

This profile configures Snakemake to run on the [SLURM Workload Manager](https://slurm.schedmd.com/) with modifications to suit
users of SCG in the Bhatt lab.

## Setup

### Deploy profile

To deploy this profile, run

	mkdir -p ~/.config/snakemake
	cd ~/.config/snakemake
	cookiecutter https://github.com/bhattlab/slurm.git

Then, you can run Snakemake with

	snakemake --profile scg ...


### Resources

The following resources are supported by on a per-rule basis:

- **mem**: set the memory resource request in mb.
- **walltime**, **runtime**: set the time resource in min.
- **ntasks**: set the number of cpus

Example rule that defines resource requirements:

	rule bwa_align:
	    input:
		"{samp}/idx/{samp}.fa".format(samp=config['sample']),
		config['reads1'],
		#config['reads2'],
		"{samp}/idx/{samp}.amb".format(samp=config['sample']),
		"{samp}/idx/{samp}.ann".format(samp=config['sample']),
		"{samp}/idx/{samp}.bwt".format(samp=config['sample']),
		"{samp}/idx/{samp}.pac".format(samp=config['sample']),
		"{samp}/idx/{samp}.sa".format(samp=config['sample'])
	    log:
		"{samp}/logs/bwa_mem.log".format(samp=config['sample'])
	    output:
		"{samp}/{samp}.bam".format(samp=config['sample'])
	    resources:
		mem=32,
		time=48,
		ntasks=24
	    shell:
		"bwa mem -t 24 {samp}/idx/{samp} {r1} {r2} ".format(
		    samp=config['sample'],
		    r1 = config['reads1'],
		    r2 = config['reads2']
		    ) +
		" | samtools sort --threads 23 > {samp}/{samp}.bam".format(samp=config['sample'])


## Tests

Test-driven development is enabled through the tests folder. Provided
that the user has installed docker and enabled [docker
swarm](https://docs.docker.com/engine/swarm/), the SLURM tests will
download two images:

1. [quay.io/biocontainers/snakemake](https://quay.io/repository/biocontainers/snakemake?tab=tags)
2. [giovtorres/docker-centos7-slurm](https://github.com/giovtorres/docker-centos7-slurm)

In addition, testing of the cookiecutter template itself is enabled
through the [pytest plugin for
Cookiecutters](https://github.com/hackebrot/pytest-cookies). You can
run the tests by issuing

	pytest -v -s
