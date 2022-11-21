---
title: "[Node.js, Express.js] Nodejs HTML to PDF"
categories:
- ExpressJS
tag:
- NodeJS
- ExpressJS
- Mustache
- Handlebars
last_modified_at: '2022-11-21 08:00:00 +0900'
---

Nodejs와 Expressjs를 활용하여 HTML파일을 PDF파일로 변경하여 문서 파일 만들기


Handlebars를 이용하여 사용할  템플릿 파일을 만듭니다.

`test.hbs`

대괄호 2개 를 사용하여 변수를 사용할 수 있습니다
```html
<html lang='ko'>
  <head>
    <meta charset='UTF-8' />
    <meta name='viewport' content='width=device-width, initial-scale=1.0' />
    <script src='https://cdn.tailwindcss.com'></script>
	</head>
	<body>
		<div class="w-full h-24">테스트 문서</div>
		</body>
</html>
```

`htmlTemplate.ts`
```js
import fs from "fs/promises"
import path from "path"
import mustache from "mustache";

export class HtmlTemplate {
  async templateFromFile(filePath: string, data: any): Promise<string> {
    const html = await fs.readFile(
      path.join(__dirname, `../pdfGenerator/templates/${filePath}.hbs`)
    );
    return this.template(html.toString(), data);
  }

  template(html: string, data: any): string {
    return mustache.render(html, data);
  }
}
```

puppeteer를 이용하여 Chromium으로 HTML파일을 랜더링해 PDF파일로 저장합니다

`pdfGenerator.ts`
```js

import puppeteer from 'puppeteer';
import { HtmlTemplate } from './htmlTemplate';
import dayjs from 'dayjs';

const puppeteerOptions = [
  '--autoplay-policy=user-gesture-required',
  '--disable-background-networking',
  '--disable-background-timer-throttling',
  '--disable-backgrounding-occluded-windows',
  '--disable-breakpad',
  '--disable-client-side-phishing-detection',
  '--disable-component-update',
  '--disable-default-apps',
  '--disable-dev-shm-usage',
  '--disable-domain-reliability',
  '--disable-extensions',
  '--disable-features=AudioServiceOutOfProcess',
  '--disable-hang-monitor',
  '--disable-ipc-flooding-protection',
  '--disable-notifications',
  '--disable-offer-store-unmasked-wallet-cards',
  '--disable-popup-blocking',
  '--disable-print-preview',
  '--disable-prompt-on-repost',
  '--disable-renderer-backgrounding',
  '--disable-setuid-sandbox',
  '--disable-speech-api',
  '--disable-sync',
  '--hide-scrollbars',
  '--ignore-gpu-blacklist',
  '--metrics-recording-only',
  '--mute-audio',
  '--no-default-browser-check',
  '--no-first-run',
  '--no-pings',
  '--no-sandbox',
  '--no-zygote',
  '--password-store=basic',
  '--use-gl=swiftshader',
  '--use-mock-keychain',
];

export const confirmationGenerateByBuffer = async (): Promise<Buffer> => {
  const browser = await puppeteer.launch({
    headless: false,
    args: puppeteerOptions
  });
  const html = await new HtmlTemplate().templateFromFile('test', {
        published_date: dayjs().format('YYYY년 MM월 DD일')
  });
  const page = await browser.newPage();
  await page.setContent(html);
  await page.evaluateHandle('document.fonts.ready');
  const pdf = await page.pdf({ format: 'A4', printBackground: true });
  await browser.close();
  return pdf;
};
```
