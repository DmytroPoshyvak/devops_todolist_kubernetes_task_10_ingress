# Інструкція з перевірки Ingress

Цей документ описує, як перевірити, що Ingress для ToDo-застосунку працює коректно.

## Що було зроблено

- Додано `Ingress` у `.infrastructure/ingress/ingress.yml`.
- Ingress має **1 HTTP-правило** з **1 path-based правилом**.
- Правило маршрутизує трафік зі шляху `/` на сервіс `todoapp-service` (порт `80`).
- Шлях захоплюється регулярним виразом `path: /(.*)` і передається в застосунок через анотацію `nginx.ingress.kubernetes.io/rewrite-target: /$1`.

## Вимоги

- Docker
- [kind](https://kind.sigs.k8s.io/)
- kubectl

## Крок 1. Створити кластер

```bash
kind create cluster --config cluster.yml
```

## Крок 2. Розгорнути застосунок та ingress-контролер

```bash
./bootstrap.sh
```

Скрипт розгортає застосунок у кластері й встановлює ingress-nginx контролер.

## Крок 3. Застосувати Ingress

```bash
kubectl apply -f .infrastructure/ingress/ingress.yml
```

## Крок 4. Перевірити ресурси

```bash
kubectl get pods -n todoapp
kubectl get svc -n todoapp
kubectl get ingress -n todoapp
kubectl describe ingress ingress -n todoapp
```

Очікуваний результат:

- усі поди в статусі `Running`;
- сервіс `todoapp-service` існує (порт `80` -> `8080`);
- Ingress `ingress` має правило з path `/(.*)` і backend `todoapp-service:80`.

## Крок 5. Перевірити доступність застосунку

```bash
curl -i http://localhost/
```

Очікується відповідь `200 OK` (або редірект `3xx`), але **не** `404`.

Відкрийте в браузері <http://localhost> — має відобразитись застосунок.

## Крок 6. Перевірити відсутність 404 у консолі браузера

1. Відкрийте <http://localhost>.
2. Відкрийте DevTools (`F12`) -> вкладка **Network** / **Console**.
3. Оновіть сторінку (`Ctrl+R` / `Cmd+R`).
4. Переконайтесь, що немає жодного запиту зі статусом `404` (статика, API, лендінг тощо).

## Автоматична перевірка

Усі кроки вище можна виконати одним скриптом:

```bash
chmod +x validate.sh
./validate.sh
```

Скрипт перевіряє поди, сервіс, Ingress та HTTP-відповіді, і завершується з ненульовим кодом, якщо щось не так.

## Видалення кластера

```bash
kind delete cluster
```