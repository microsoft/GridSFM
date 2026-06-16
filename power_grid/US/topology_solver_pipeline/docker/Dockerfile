# =============================================================================
# Dockerfile — topology_solver_pipeline: from raw topology to solved GridSFM scenarios
# =============================================================================
#
# This image packages the full topology_solver_pipeline (stages 2-3) from the
# GridSFM project. It installs Julia 1.11, all required Julia
# packages (PowerModels.jl, Ipopt, etc.), and provides entry points for
# each pipeline stage:
#
#   Stage 2: solve_topo_json.jl         — make a raw topology cold-strict solvable
#   Stage 3: export_gridsfm_data.jl     — export solvable to gridSFM .pyg.json
#            gen_perturbed_data.jl      — bulk perturbed scenario generation
#   Verify:  solve_pyg_json.jl          — round-trip verify a .pyg.json
#   Test:    integration_test_all_components.sh — full end-to-end smoke test
#
# Build (from topology_solver_pipeline/):
#   docker build -t topology_solver_pipeline -f docker/Dockerfile .
#
# Run (mount your data):
#   docker run --rm -v /path/to/data:/data -v /path/to/output:/output \
#       topology_solver_pipeline -c 'julia --project=/app solve_topo_json.jl \
#       /data/16h/alabama_model.json /output/alabama.solvable.json'
#
# See README.md for detailed usage instructions.
# =============================================================================

# --- Base image: Julia 1.11 on Debian bookworm ---
FROM julia:1.12-bookworm

LABEL maintainer="Microsoft Research"
LABEL description="topology_solver_pipeline — power grid topology to OPF-solved gridSFM scenarios"

# Install system dependencies (Python3 for consistency checks, basic tools)
RUN apt-get update && apt-get install -y --no-install-recommends \
        python3 \
        python3-pip \
        bash \
        grep \
        coreutils \
        procps \
    && rm -rf /var/lib/apt/lists/*

# --- Copy pipeline code ---
WORKDIR /app
COPY Project.toml Manifest.toml /app/
COPY run_opf_relaxation.jl /app/
COPY solve_topo_json.jl /app/
COPY export_gridsfm_data.jl /app/
COPY gen_perturbed_data.jl /app/
COPY solve_pyg_json.jl /app/
COPY run_gen_gridsfm_data.sh /app/
COPY integration_test_all_components.sh /app/
COPY shared/ /app/shared/
COPY docker/ /app/docker/

# --- Install Julia packages (pinned by Manifest.toml) ---
# This is the slow step — cached by Docker layer unless Project/Manifest change
RUN julia --project=/app -e 'using Pkg; Pkg.instantiate(); Pkg.precompile()'

# --- Verify packages load correctly ---
RUN julia --project=/app -e 'using PowerModels, Ipopt, JuMP, JSON; println("✓ All packages loaded")'

# --- Create mount points for data I/O ---
RUN mkdir -p /data /output

# --- Default working directory ---
WORKDIR /app

# --- Default entrypoint: bash for flexibility ---
ENTRYPOINT ["/bin/bash"]
