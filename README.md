# Канбан — доска задач (PWA)

Приложение «Канбан»: доски задач, дедлайны, события, замер времени, чек-листы,
подзадачи, метки, вложения, комментарии, отчёты и графики.

- Устанавливается на главный экран Android и iPhone (PWA)
- Аккаунты через Google, у каждого пользователя свои данные, доступные с любого устройства
- Фронтенд: GitHub Pages · Бэкенд: Supabase (бесплатный тариф)

## Настройка бэкенда (один раз)

1. Зарегистрируйтесь на https://supabase.com и создайте новый проект
   (Free-тариф), запишите пароль от базы.

2. В панели проекта слева: **Authentication → Providers → Google → Enable**.
   Там будет показан Redirect URL вида
   `https://<ваш-проект>.supabase.co/auth/v1/callback` — скопируйте его.

3. Создайте OAuth-клиент Google: https://console.cloud.google.com/apis/credentials
   - «Создать учётные данные» → «OAuth client ID»
   - Тип: «Веб-приложение»
   - **Authorized redirect URIs** добавьте:
     - `https://<ваш-проект>.supabase.co/auth/v1/callback`
     - `https://<ваш-логин>.github.io/kanban-pwa`
   - Нажмите «Создать», скопируйте Client ID и Client Secret.

4. Вернитесь в Supabase: **Authentication → Providers → Google**:
   - Вставьте Client ID и Client Secret, включите «Enable sign in».
   - В Site URL (Authentication → URL Configuration) укажите
     `https://<ваш-логин>.github.io/kanban-pwa`.

5. Создайте таблицу для данных. Откройте **SQL Editor** и выполните:

```sql
create table if not exists profiles (
  user_id uuid primary key,
  data jsonb not null,
  updated_at timestamptz not null default now()
);
alter table profiles enable row level security;
create policy "own profile" on profiles
  for all using (auth.uid() = user_id)
  with check (auth.uid() = user_id);
alter table profiles replica identity full;
```

6. Возьмите ключи: **Project Settings → API**
   - Project URL (например `https://xxxx.supabase.co`)
   - `anon` public key

7. В файле `index.html` найдите блок `BACKEND` в начале скрипта и вставьте:

```js
const BACKEND={url:'https://xxxx.supabase.co',anonKey:'ВАШ_ANON_КЛЮЧ',enabled:true};
```

8. Сохраните и запушите изменения — сайт на GitHub Pages обновится.

## Развёртывание на GitHub Pages

```sh
git init && git add -A && git commit -m "Канбан PWA"
gh repo create kanban-pwa --public --source=. --push
gh api -X POST repos/otdelroznica-boop/kanban-pwa/pages \
  -f "source[branch]=main" -f "source[path]=/"
```

Сайт будет на https://otdelroznica-boop.github.io/kanban-pwa/

## Установка на телефон

- **Android:** откройте сайт в Chrome → «⋮» → «Добавить на главный экран».
- **iPhone:** откройте в Safari → «Поделиться» → «На экран «Домой»».
