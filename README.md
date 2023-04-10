# Laravel Schedule

Trabalhando com tarefas agendadas no Laravel.

## Tutorial

-   Onde fica os schedule, tarefas agendadas do Laravel?<br>
    No arquivo Kernel.php em [app/Console/Kernel.php](app/Console/Kernel.php)

-   Link da documentação: [clique aqui](https://laravel.com/docs/10.x/scheduling)

-   Exemplo de Agendamento de Tarefa

```php
$schedule->call(function () {
    DB::table('recent_users')->delete();
})->daily();
```

-   Precisa ter o metodo `__invoke`

```php
$schedule->call(new DeleteRecentUsers)->daily();
```

-   Lista de tarefas agendadas

```sh
php artisan schedule:list
```

-   Chamando job agendado

```php
$schedule->job(new Heartbeat)->everyFiveMinutes();
```

-   Executar comando no sistema operacional

```php
$schedule->exec('node /home/forge/script.js')->daily();
```

-   Executar semanalmente, toda segunda-feira às 13:00

```php
    $schedule->call(function () {
    // ... jobs, commands...
    })->weekly()->mondays()->at('13:00');
```

-   Rodar de hora em hora, entre as 08:00 e 17:00 pelo fuso horário de Chicago

```php
$schedule->command('foo')
->weekdays()
->hourly()
->timezone('America/Chicago')
->between('8:00', '17:00');
```

-   Executar de hora em hora apenas entre as 07 e 22

```php
$schedule->command('emails:send')
->hourly()
->between('7:00', '22:00');
```

-   Evitar sobreposiçao de comandos

```php
$schedule->command('emails:send')->withoutOverlapping(10);
```

-   Bloquei para executar em só servidor. Driver de cache deve estar sendo compartilhado e acesso entre os servidores

```php
$schedule->command('report:generate')
->fridays()
->at('17:00')
->onOneServer();
```

-   Rodar em backaground, sem travar o schedule, agendamentos de commands e exec

```php
$schedule->command('analytics:report')
->daily()
->runInBackground();
```

-   Configurar Cron

```text
* * * * * cd /path-to-your-project && php artisan schedule:run >> /dev/null 2>&1
```

-   Executar Schedule localmente

```sh
php artisan schedule:work
```

## Cron

### Sem Laravel Sail

-   Instalar o cron

```sh
    apt-get update -y \
    && apt-get install cron -y \
    && echo "* * * * * cd /var/www/html && php artisan schedule:run >> /dev/null 2>&1" >> /etc/cron.d/scheduler \
    && chmod 644 /etc/cron.d/scheduler \
    && crontab /etc/cron.d/scheduler
```

-   Certifique que o cron está em execução no servidor/ambiente, caso não esteja inicie-o ou então configure o supervisor para cuidar disso.

### Com Laravel Sail

-   Se estiver utilizando o Laravel Sail é necessário publicar o Laravel Sail primeiro, para depois manuzear a imagem docker em /docker

```sh
    php artisan sail:publish
    # sail artisan sail:publish
```

-   Encontre a versão do php que está sendo utilizado pelo Laravel Sail em `./docker` e então no Dockerfile correspondente, no fim do arquivo antes de `EXPOSE 8000`, em uma nova linha, adicione o seguinte comando:

```dockerfile
RUN apt-get update -y \
    && apt-get install cron -y \
    && echo "* * * * * cd /var/www/html && php artisan schedule:run >> /dev/null 2>&1" >> /etc/cron.d/scheduler \
    && chmod 644 /etc/cron.d/scheduler \
    && crontab /etc/cron.d/scheduler
```

-   Adicione o serviço do cron ao supervisor, para isso vá em `./docker/?/supervisord.conf` e adicone as seguintes linhas ao fim do arquvio:

```text
[program:cron]
command=/usr/sbin/cron -f -l 8
autostart=true
stdout_logfile=/var/log/cron.out.log
stderr_logfile=/var/log/cron.err.log
```

-   Como o Dockerfile foi modificado, precisamos realizar novo build da imagem docker, por isso execute:

```sh
sail build
# em caso de problema de cache rode: sail build --no-cache
```
