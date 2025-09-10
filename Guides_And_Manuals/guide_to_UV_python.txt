UV :

INTRODUCTION:
-----
- package manager for python 
- 10x faster because of its parallel computation
- manages your memory space efficiently while also taking care of proper structure 

INSTALLATION GUIDE:
-------------------
For Windows:
------------
There are couple of ways you could have this installed in your system.
1. using irm -> 
powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/0.8.13/install.ps1 | iex"
2. using PyPi ->
pipx install uv // pip install uv
3. Other options sited in link : https://docs.astral.sh/uv/getting-started/installation

For Linux:
----------
check if your system itself has it 
link : https://docs.astral.sh/uv/getting-started/installation 

UPGRADATION GUIDE:
------------------
Once installed, use : uv self update 
Updating uv will re-run the installer and can modify your shell profiles. To disable this behavior, set UV_NO_MODIFY_PATH=1. 


UNINSTALLATION: 
--------------
check link : https://docs.astral.sh/uv/getting-started/installation/#uninstallation 




-wsl -> check version for windows (make sure it is updated) then start doing it there
ak