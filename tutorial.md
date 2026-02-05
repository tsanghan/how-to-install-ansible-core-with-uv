# How to install Ansible-Core with UV - Part 1

uv and uvx will be installed in $HOME/.local/bin, we will not be using uvx in this session.

```bash
cd $HOME
curl -LsSf https://astral.sh/uv/install.sh | sh
source ~/.bashrc
```

Check your system python version

```bash
python --version
```

Create a Project directory
```bash
mkdir -p $HOME/Projects/project1
cd $HOME/Projects/project1
```

To view available and installed Python versions
```bash
uv python list
```

Install a specific python version and pin project python version
```bash
uv python pin 3.13.12
```

initiate a bare project directory, since we are not doing software development
```bash
uv init --bare
```

Create a venv in project directory
```bash
uv venv --python 3.13.12
```

Activate project virtual environment and take note of the change in your shell prompt
```bash
source .venv/bin/activate
```

Check project python version
```bash
python --version
```

Install ansible-core
```bash
uv add ansible-core
```

Check ansible version and take note of the value for the line "config file"
```bash
ansible --version
```

Create an ansible.cfg file
```bash
cat <<EOF | tee ansible.cfg
[defaults]
inventory = ./inventory
roles_path = ./roles
retry_files_enabled = False
host_key_checking = False
collections_path = ./collections
interpreter_python = auto_silent
deprecation_warnings = False
EOF
```

Re-check ansible version and take note of the value for the line "config file" again
```bash
ansible --version
```

Test ansible with localhost ping
```bash
ansible localhost -m ping
```

Install ansible-lint
```bash
uv add ansible-lint
```

Check ansible-lint version
```bash
ansible-lint --version
```

Install F5 BIG-IP Imperative Collection for Ansible
```bash
ansible-galaxy collection install f5networks.f5_modules
```

Check F5 BIG-IP Imperative Collection for Ansible version
```bash
ansible-galaxy collection list
```

Exit from virtual environment and take note of the change in your shell prompt
```bash
deactivate
```

# How to improve workflow - Part 2

Can we automate the activate/deactivate worflow?

Yes we can with *direnv*

Change directory back to $HOME
Install *direnv*.\
*direnv* will be installed in $HOME/.local/bin

```bash
cd $HOME
curl -sfL https://direnv.net/install.sh | bash
```

Hook direnv into your shell by adding the following line at the end of the ~/.bashrc file
```bash
eval "$(direnv hook bash)"
```

To activate the above hook in your current shell
```bash
source ~/.bashrc
```

Currently direnv does not integrate with uv, we will create an extention for the automation of invoking uv utility
```bash
mkdir -p /home/${USER}/.config/direnv
cat <<'EOF' > /home/${USER}/.config/direnv/direnvrc
layout_uv() {
    if [[ -d ".venv" ]]; then
        VIRTUAL_ENV="$(pwd)/.venv"
    fi

    if [[ -z $VIRTUAL_ENV || ! -d $VIRTUAL_ENV ]]; then
        log_status "No virtual environment exists. Executing \`python -m venv .venv\`."
        python -m venv .venv
        VIRTUAL_ENV="$(pwd)/.venv"
    fi

    # Activate the virtual environment
    . $VIRTUAL_ENV/bin/activate
}
EOF
```

Create a second project directory
```bash
mkdir $HOME/Projects/project2
```
direnv will automatically activate virtual environment when we change direcotry (cd) into the project directory
Currently, we have not set up the project2 directory
We will not set up the project2 directory
And we want to use Python 3.14.3 for this project
```bash
uv python install 3.14.3
pushd $HOME/Projects/project2
uv python pin 3.14.3
uv init --bare
uv venv --python 3.14.3
popd
echo "layout uv" | tee $HOME/Projects/project2/.envrc
```

Now we cd into $HOME/Projects/project2
```bash
cd $HOME/Projects/project2
```

You will get a message "direnv: error /home/tsanghan/Projects/project2/.envrc is blocked. Run `direnv allow` to approve its content"
Type the following command to allow
```bash
direnv allow
```

Notice the virtual environment is automatically activated, even the prompt does not show
```bash
python --version
```

Follow instrunction steps from part 1 to install
1) ansible-core
2) ansible-lint
3) F5 BIG-IP Imperative Collection for Ansible
4) do remember to create a project *ansible.cfg* file

To deactivate the virtual environment, simply cd out of the project direcotry.

```bash
cd ..
```

To have promp show your virutal environment, paste the following to the end of ~/.bashrc
```bash
show_virtual_env() {
  if [[ -n "$VIRTUAL_ENV_PROMPT" && -n "$DIRENV_DIR" ]]; then
    echo "($(basename $VIRTUAL_ENV_PROMPT)) "
  fi
}
PS1='$(show_virtual_env)'$PS1
```

Source the file ~/.bashrc

Cd into project2 directoory, take note of the prompt.