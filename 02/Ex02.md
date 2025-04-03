# Budowanie aplikacji w kontenerze

## Stworzenie przykładowej aplikacji .net core

```bash
dotnet new mvc -n mydockerapp

cd mydockerapp

# Run the project
dotnet run

# Open the website in the browser
```

## Będziemy chcieli przenieść poniższy proces do Dockerfile

```bash
dotnet restore "mydockerapp.csproj"
dotnet build "mydockerapp.csproj" -c Release -o /out
dotnet publish "mydockerapp.csproj" -c Release -o /app

dotnet mydockerapp.dll
```

# Kroki

## Krok 1. Zbudowanie aplikacji w kontenerze

Pierw stwórz plik `Dockerfile` w obecnym katalogu.

Do tego zadania przydadzą Ci się dwa zestawy poleceń:

### Polecenia pliku Dockerfile:
- `FROM XXX` - określa warstwę z której korzystamy. Gdzie XXX jest docker image. Do budowania wykorzystamy obraz `mcr.microsoft.com/dotnet/sdk:6.0`
- `COPY [ "local_source", "container_dest" ]` - kopiuje pliki lokalne do kontenera. Nawiasy są ważne
- `WORKDIR` - określa katalog roboczy (polecenia będą wykonywane względem tego katalogu)
- `RUN` - wykonuje polecenie w konsoli wewnątrz kontenera

Polecenia dotnet core: 
- `dotnet restore "XXX.csproj"` - restore paczek dotnet core
- `dotnet build "XXX.csproj" -c Release -o /YYY` - zbudownanie projektu
    Legenda:
    - `XXX.csproj` - jest ścieżką do pliku csproj
    - `YYY` - jest katalogiem gdzie chcemy umieścić zbudowane artefakty

Komendy w konsoli:

- `docker build . -t MYTAG` - zbuduje plik Dockerfile istniejący w aktualnym katalogu i nada mu tag MYTAG
- `docker images` - wyświetla wszystkie obrazy w repozytorium
- `docker images XXX` - wyświetla wszystkie obrazy o nazwie XXX

# Krok 2. Publikacja aplikacji w kontenerze

Polecenia w Dockerfile:
- `FROM source_image AS name` - dodanie AS po nazwie obrazu umożliwia posługiwanie się nazwą własną docker image.

Polecenia dotnet core:
- `dotnet publish "XXX.csproj" -c Release -o /app` - przygotowuje paczkę dotnet core gotową do publikacji 

# Krok 3. Uruchomienie aplikacji w kontenerze

Stwórz kontener który skorzysta z opublikowanej paczki i uruchomi naszą aplikację dotnet core

Komendy/informacje Dockerfile:
- `EXPOSE 80` - wystawia port 80 na zewnątrz
- Obraz do uruchamiania: `mcr.microsoft.com/dotnet/aspnet:5.0-buster-slim` - poprzedni to było SDK.
- `COPY --from=XXX  /app .` - `--from=XXX` gdzie `XXX` jest nazwą docker image z którego będziemy wykonywali kopiowanie
- `ENTRYPOINT [“exec”, “parameters”]` - określa jaka komenda ma zostać uruchomiona przy wykonaniu docker run 

Komendy dotnet core:
- `dotnet XXX.dll` - uruchamia aplikacje dotnet

Komendy w konsoli:
- `docker run --rm -p LOCAL_PORT:DOCKER_PORT --name CONTAINER_NAME DOCKER_IMAGE` - tworzy i uruchamia instancję kontenera dockerowego.

Gdze:
- `--rm` - po zatrzymaniu kontener zostanie automatycznie usunięty
- `-p `- mapowanie portów
- `--name` - nadanie własnego name uruchamianemu kontenerowi
