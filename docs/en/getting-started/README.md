# Getting started :id=intro :priority=9
This article describes simplified installation using Docker. For standalone installation, check «[Advanced installation](/en/advanced/)» guide.

## Minimal requirements  :id=requirements
* RAM: at least 2Gb
* Storage: at least 5Gb of reserved disk space
* Docker: >= 18.09

## Installation  :id=installation

Execute commands below in Terminal and follow installation wizard's instructions
```bash
wget https://dl.cattr.app/installer/cattr.sh
bash cattr.sh
```

If you want to update your Cattr instance execute commands below in Terminal at the server, where Cattr was installed
```bash
wget https://dl.cattr.app/installer/updater.sh
bash updater.sh
```

## Common errors list  :id=errors

<details>
<summary>
docker: Cannot connect to the Docker daemon at unix:///var/run/docker.sock. Is the docker daemon running?
</summary>

**Make sure Docker is installed on the server you're trying to launch Cattr, and execute the command one more time**
</details>

<details>
<summary>
Error starting userland proxy: listen tcp 0.0.0.0:80 bind: address already in use.
</summary>

**Make sure Cattr is not running already, and there is no other web services installed on the server you're trying to launch Cattr on. We would suggest you consulting your system administrator for that matter.**
</details>

?> If you bumped into error that wasn't described above, feel free to ask a question in our [community](https://community.cattr.app).

## What's next?  :id=next

After you finish installing and configuring the Frontend and Backend modules, you will be able to login with the credentials you provided for Administrator user. Once you log in, you'll be able to create projects, tasks, and assign them to the new users.

?>You can read about how to do it all in [Create project](ru/workflow/?id=project), [Create task](ru/workflow/?id=task) and [Create user](ru/users/?id=create) sections.
