# Этап сборки
FROM mcr.microsoft.com/dotnet/sdk:10.0 AS build

WORKDIR /src

# Установка Node.js и Yarn для frontend
RUN curl -fsSL https://deb.nodesource.com/setup_20.x | bash - \
    && apt-get install -y nodejs \
    && npm install -g yarn \
    && rm -rf /var/lib/apt/lists/*

# Клонирование ветки v5-develop
RUN apt-get update && apt-get install -y git \
    && git clone --depth 1 --branch v5-develop-pariv https://github.com/pariv/Sonarr.git . \
    && rm -rf /var/lib/apt/lists/*

# Сборка frontend
WORKDIR /src/frontend
RUN yarn install && yarn build

# Сборка backend
WORKDIR /src
RUN dotnet msbuild -restore src/Sonarr.sln \
    -p:Configuration=Release \
    -p:Platform=Posix \
    -t:PublishAllRids

# Финальный этап
FROM mcr.microsoft.com/dotnet/aspnet:10.0

WORKDIR /app

# Установка runtime зависимостей
RUN apt-get update && apt-get install -y \
    mediainfo \
    sqlite3 \
    libicu-dev \
    curl \
    && rm -rf /var/lib/apt/lists/*

# Копирование собранных файлов
# Путь может отличаться, проверьте структуру _output после сборки
COPY --from=build /src/_output/net8.0/linux-x64/publish /app
COPY --from=build /src/_output/UI /app/UI

VOLUME /config
EXPOSE 8989

ENTRYPOINT ["/app/Sonarr", "-nobrowser", "-data=/config"]
