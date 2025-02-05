# Többnyelvű Angular program

* Szerző: Sallai András
* Licenc: [CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/)
* Copyright @ 2025, Sallai András

## Angular projekt létrehozása

```bash
ng new app01
cd app01
code .
```

## Függőségek telepítése

```bash
npm install @ngx-translate/core
npm install @ngx-translate/http-loader
```

## Beállítások

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

app.component.html:

```html

<select (change)="translateText($event)">
  <option value="en">English</option>
  <option value="hu" selected>Magyar</option>
</select>

<div>{{ 'app.hello' | translate}}</div>

<app-home></app-home>
```

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
  title = 'app01';

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

## A Home komponens

```bash
ng g c home
```

home.component.html:

```html
<p>home works!</p>

<div>{{ 'home.title' | translate}}</div>
```

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

```bash
npm install @vendure/ngx-translate-extract --save-dev
```

```json
"scripts": {
  "extract": "ngx-translate-extract --input ./src --output ./src/strings.json --format json"
},
```

```bash
npm run extract
```
