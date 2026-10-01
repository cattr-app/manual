# Документация для разработчиков

## Сборка настольного клиента Cattr :id=cattr-client-build

Настольный клиент Cattr — приложение Electron для Windows, macOS и Linux. Текущий набор нативных зависимостей требует указанных ниже версий Node.js и npm.

### Требования для сборки

- x64 Windows, macOS или Linux
- Node.js 14.21.x
- npm 9.9.4
- Python 3.10 и нативный toolchain для C/C++
- Git

В macOS установите Xcode с [сайта Apple для разработчиков](https://developer.apple.com/xcode/). В Debian/Ubuntu установите системные зависимости для сборки:

~~~bash
sudo apt-get update
sudo apt-get install -y git cmake curl python3 build-essential pkg-config \
  libsecret-1-0 libsecret-1-dev ca-certificates openssh-client dpkg-dev dpkg-sig
~~~

В Windows установите Python 3.10 и Visual Studio 2022 Build Tools с компонентом Desktop development with C++. Для нативной сборки Windows Docker не нужен.

Установите Node.js 14.21.x с помощью менеджера версий, например [nvm](https://github.com/nvm-sh/nvm), затем установите используемую проектом версию npm:

~~~bash
nvm install 14.21
nvm use 14.21
npm install --global npm@9.9.4
~~~

### Получение исходного кода и запуск в режиме разработки

~~~bash
git clone https://github.com/cattr-app/desktop-application.git
cd desktop-application
npm ci
npm run build-development
npm run dev
~~~

В Windows вместо npm run dev используйте npm run dev-win. Режим разработки хранит данные отдельно от обычного профиля клиента.

### Сборка production-пакетов

Укажите версию приложения, соберите renderer и подготовьте пакет для текущей платформы:

~~~bash
npm ci
npm --no-git-tag-version version 1.0.0
npm run build-production
npm run package-linux
~~~

Выберите команду для целевой платформы:

| Платформа | Команда | Результат |
| --- | --- | --- |
| macOS, с подписью и нотариальным заверением | npm run package-mac | DMG; нужны учетные данные Apple |
| macOS, без подписи | npm run package-mac-unsigned | DMG |
| Linux | npm run package-linux | AppImage, DEB и tar.gz |
| Windows | npm run package-windows | NSIS-установщик и portable-приложение |

Пакеты сохраняются в target/. Пакеты macOS нужно собирать на macOS. Linux также может собирать пакеты Windows при установленном Wine; Windows собирает пакеты только для Windows.
