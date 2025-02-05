# Többnyelvű Angular program

* Szerző: Sallai András
* Licenc: [CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/)
* Copyright @ 2025, Sallai András

## Angular projekt létrehozása

Hozzunk létre egy Angular projektet. Lépjünk be a könyvtárba, majd indítsuk el a VSCode-t.

```bash
ng new app01
cd app01
code .
```

Vegyünk fel egy home komponenst is, amit később használni fogunk.

```bash
ng g c home
```

## Függőségek telepítése

VSCode-ban egy terminálban telepítsük a függőségeket:

```bash
npm install @ngx-translate/core
npm install @ngx-translate/http-loader
```

## Beállítások

Az app.config.ts fájlban fel kell vennünk egy új függvényt:

```typescript
export function createTranslateLoader(http: HttpClient) {
  return new TranslateHttpLoader(http, './i18n/', '.json');
}
```

Két újabb privider-t kell beállítani:

```typescript
    importProvidersFrom([
      TranslateModule.forRoot({
        loader: {
            provide: TranslateLoader,
            useFactory: (createTranslateLoader),
            deps: [HttpClient]
        }
      })
    ]),
    provideHttpClient(withInterceptorsFromDi())
```

Importáljuk a függőségeket. A teljeskódot az alábbikaban láthatjuk.

app.config.ts:

```typescript
import { ApplicationConfig, importProvidersFrom, provideZoneChangeDetection } from '@angular/core';
import { provideRouter } from '@angular/router';
import { routes } from './app.routes';
import { 
  HttpClient,
  provideHttpClient,
  withInterceptorsFromDi
} from '@angular/common/http';
import { TranslateHttpLoader } from '@ngx-translate/http-loader';
import { TranslateLoader, TranslateModule } from '@ngx-translate/core';


export function createTranslateLoader(http: HttpClient) {
  return new TranslateHttpLoader(http, './i18n/', '.json');
}

export const appConfig: ApplicationConfig = {
  providers: [
    provideZoneChangeDetection({ eventCoalescing: true }), 
    provideRouter(routes),
    importProvidersFrom([
      TranslateModule.forRoot({
        loader: {
            provide: TranslateLoader,
            useFactory: (createTranslateLoader),
            deps: [HttpClient]
        }
      })
    ]),
    provideHttpClient(withInterceptorsFromDi())
  ]
};
```

## Az app komponens

Az app.component.ts fájlban @Component dekorátor import kulcsához fel kell venni a TranslateModule-t.

```typescript
  imports: [
    TranslateModule,
    HomeComponent
  ],
```

Szükségünk van egy translate nevű objektumra, amivel beállíthatjuk az aktuális nyelvet.

```typescript
  translate!: TranslateService;
```

Be kell fecskendezni a TranslateService-t:

```typescript
  constructor(private translateService: TranslateService) {}
```

Inicializálni kell a translate objetumot az ngOnInit() metódusban:

```typescript
  ngOnInit(): void {
    this.translate = this.translateService;
    this.translate.setDefaultLang('hu');
  }
```

Szükségünk lesz egy metódusra, ami beállítja az aktuális nyelvet:

```typescript
  translateText(lang: any) {
    this.translate.use(lang.target.value);
  }
```

A teljes app.component.ts fájlt az alábbiakban látjuk:

app.component.ts:

```typescript
import { Component } from '@angular/core';
import { TranslateModule, TranslateService } from '@ngx-translate/core';
import { HomeComponent } from "./home/home.component";

@Component({
  selector: 'app-root',
  standalone: true,
  imports: [
    TranslateModule,
    HomeComponent
  ],
  templateUrl: './app.component.html',
  styleUrl: './app.component.css'
})
export class AppComponent {

  translate!: TranslateService;

  constructor(private translateService: TranslateService) {}

  ngOnInit(): void {
    this.translate = this.translateService;
    this.translate.setDefaultLang('hu');
  }

  translateText(lang: any) {
    this.translate.use(lang.target.value);
  }
}
```

A sablon állományban (.html fájl) felveszünk egy legördülő listadobozt, amiből válaszhatunk nyelvet. Legyen valami div elem is amiben egy "Helló Világ" szöveget tervezünk.

A teljes kód:

app.component.html:

```html

<select (change)="translateText($event)">
  <option value="en">English</option>
  <option value="hu" selected>Magyar</option>
</select>

<div>{{ 'app.hello' | translate}}</div>

<app-home></app-home>
```

## A Home komponens

A home komponensben is vegyünk fel legalább egy szöveget.

home.component.html:

```html
<p>home works!</p>

<div>{{ 'home.title' | translate}}</div>
```

A home komponens TypeScript állományában is fel kell vagyük a translate objektumot, és inicializálni kell.

Teljes kód:

home.component.ts:

```typescript
import { Component } from '@angular/core';
import { TranslateModule, TranslateService } from '@ngx-translate/core';

@Component({
  selector: 'app-home',
  standalone: true,
  imports: [TranslateModule],
  templateUrl: './home.component.html',
  styleUrl: './home.component.css'
})
export class HomeComponent {
  translate!: TranslateService;

  constructor(private translateService: TranslateService) {}

  ngOnInit(): void {
    this.translate = this.translateService;    
  }
}
```

## Kulcsok kiszedése

A forrásfájlokból ki kell gyűjtenünk a kulcsokat. De mik azok a kulcso?

Vegyük elő az app.component.html fájlt. Egy ilyen sort tettünk bele:

```html
<div>{{ 'app.hello' | translate}}</div>
```

Ebben a sorban a kulcs:

* app.hello

Ezeket a kulcsokat kell kigyüjtenünk, mivel ezek mellé fogjuk írni a fordításokat. A kulcsok JSON fájlokba fogjuk gyűjteni. Elsőként az src könyvtárba készítünk egy strings.json nevű állományt. Kézzel is megtehetjük, de a következő csomag segíti ezt a munkát. 

Telepítsük a @vendure/ngx-translate-extract csomagot: 

```bash
npm install @vendure/ngx-translate-extract --save-dev
```

Futtathatjuk parancssorból vagy írhatunk egy scriptet a package.json fájlban:

```json
"scripts": {
  "extract": "ngx-translate-extract --input ./src --output ./src/strings.json --format json"
},
```

A package.json fájlba írt script futtatása:

```bash
npm run extract
```

## Fordítás

Az src/strings.json fájlt másoljuk le kétszer. Egy angol és egy magyar nyevli fájl hozunk létre a public/i18n/ könyvtárban. A public könyvtárban hozzuk létre a i18n könyvtárat is, mivel az alapértelmezetten nem létezik.

Az angol nyelvnek egy en.sjon, a magyar nyelven egy hu.json nevű fájlt hozzunk létre:

* public/i18n/en.json
* public/i18n/hu.json

public/i18n/en.json:

```json
{
  "app.hello": "Hello World",
  "home.title": "Home"
}
```

public/i18n/hu.json:

```json
{
  "app.hello": "Helló Világ",
  "home.title": "Főoldal"
}
```

Ha újabb kulcsot hozok létre valamelyik .html fájlban, azt itt is fel kell venni.
Innentől használható a többnyelvű program.
