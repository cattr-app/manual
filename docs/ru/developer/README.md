# Документация для разработчиков

## Сборка клиентского приложения Cattr :id=cattr-client-build

Клиентское приложение Cattr для рабочего стола основано на фреймворке Electron. 
Запуск приложения тестировался на следующих операционных системах на CPU x86_x64:

- MacOS: Monterey 12.3.1  
- Windows: 22H2 10.0.19045, 11.0.22621
- Debian: bullseye+kde 11
- Ubuntu: LTS 22.04
- Alt linux: kworkstation
- Astra linux: orel 2.12

### Для успешной сборки вам потребуются следующие зависимости:

#### Для MacOS
Вам необходимо установить xcode с [официального сайта](https://developer.apple.com/xcode/)

#### Для Linux (apt based)
```bash
apt-get update
apt-get install -y git cmake curl python3 build-essential pkg-config libsecret-1-0 libsecret-1-dev ca-certificates openssh-client dpkg-dev dpkg-sig
```
##### Установка nodejs 14.19.0 (MacOS & Linux)  
Проще всего это сделать с помощью nvm, вот [официальное руководство по установке](https://github.com/nvm-sh/nvm?tab=readme-ov-file#install--update-script).  

Теперь мы можем использовать его для установки nodejs.  
```bash
nvm install 14.19.0
nvm use 14.19.0
```
Установка yarn
```bash
npm install -g yarn
```

Вы можете проверить установку следующим образом:
```bash
node -v # v14.19.0
yarn -v # 3.2.1
```

#### Windows
##### Скачайте и установите Docker Desktop с [официального сайта](https://www.docker.com/).


![docker](../../assets/en/getting-started/docker.png)

Для работы Docker в Windows вам может потребоваться включить виртуализацию в BIOS и [установить WSL 2](https://learn.microsoft.com/en-us/windows/wsl/install). Процесс установки подробно описан [в руководстве пользователя Docker](https://docs.docker.com/desktop/setup/install/windows-install/).


## Запуск версии для разработки (только Linux & MacOS)
1. Клонируйте этот репозиторий [https://git.amazingcat.net/cattr/desktop/desktop-application/](https://git.amazingcat.net/cattr/desktop/desktop-application/) и откройте его директорию
2. Установите зависимости через `yarn`
3. Укажите версию, например `v1.0.0"`
```bash
npm config set git-tag-version false
npm version v1.0.0
```
4. Запустите webpack через `yarn build-development` для версии разработки
5. После завершения сборки запустите `yarn dev` для запуска клиента в режиме разработки

## Режим разработки
Установка для разработки использует другое имя службы связки ключей и другой путь к папке приложения (с суффиксом "-develop").

## Сборка production версии
1. Клонируйте этот [https://git.amazingcat.net/cattr/desktop/desktop-application/](https://git.amazingcat.net/cattr/desktop/desktop-application/) репозиторий и откройте его директорию
2. (Только Windows) запустите в PowerShell `docker run -it -v ${PWD}:/project electronuserland/builder:14-wine` следующие команды должны быть выполнены внутри запущенного контейнера.
3. Установите зависимости через `yarn`
4. Укажите версию, например `v1.0.0`
```bash
npm config set git-tag-version false
npm version v1.0.0
```
5. Соберите приложение в production режиме через `yarn build-production`
6. Соберите исполняемый файл для вашей платформы (выходная директория `/target`).


Как собрать исполняемый файл?
  - **macOS:** `yarn package-mac` создаст подписанный и нотариально заверенный DMG
  - **Linux:** `yarn package-linux` создаст Tarball, DPKG и AppImage
  - **Windows:** `yarn package-windows` создаст установщик и портативные исполняемые файлы

Таблица совместимости:
  - **Хост с macOS:** может создавать сборки только для macOS
  - **Хост с Linux:** может создавать сборки для Linux и Windows (используя Wine)
  - **Хост с Windows:** может создавать сборки только для Windows
