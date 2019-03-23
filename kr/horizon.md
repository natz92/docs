# Laravel Horizon
# 라라벨 Horizon

- [Introduction](#introduction)
- [시작하기](#introduction)
- [Installation](#installation)
- [설치하기](#installation)
    - [Configuration](#configuration)
    - [설정하기](#configuration)
    - [Dashboard Authorization](#dashboard-authorization)
    - [Dashboard 권한 부여](#dashboard-authorization)
- [Running Horizon](#running-horizon)
- [Horizon 실행하기](#running-horizon)
    - [Deploying Horizon](#deploying-horizon)
    - [Horizon 배포하기](#deploying-horizon)
- [Tags](#tags)
- [태그](#tags)
- [Notifications](#notifications)
- [알림](#notifications)
- [Metrics](#metrics)
- [메트릭](#metrics)

<a name="introduction"></a>
## Introduction
## 시작하기

Horizon provides a beautiful dashboard and code-driven configuration for your Laravel powered Redis queues. Horizon allows you to easily monitor key metrics of your queue system such as job throughput, runtime, and job failures.

Horizon은 Redis Queue를 사용하는 라라벨을 위해서 아름다운 대시보드 와 코드를 기반으로한 설정기능을 제공합니다. Horizon을 사용하면 job의 처리량, 실행시간 및 실패한 job과 같은 Queue 시스템의 주요 메트릭을 손쉽게 모니터링 할 수 있습니다.

All of your worker configuration is stored in a single, simple configuration file, allowing your configuration to stay in source control where your entire team can collaborate.

모든 worker의 설정은 하나의 간단한 설정 파일에 저장되기 때문에, 팀원 모두와 협업 할 수 있도록 소스 컨트롤에 보관 할 수 있습니다.

<p align="center">
<img src="https://res.cloudinary.com/dtfbvvkyp/image/upload/v1537195039/photos/Test.png" width="600" height="481">
</p>

<a name="installation"></a>
## Installation
## 설치하기

> {note} You should ensure that your queue driver is set to `redis` in your `queue` configuration file.

> {note} 큐 드라이버는 `queue` 설정 파일에서 `redis` 로 설정되어야합니다.

You may use Composer to install Horizon into your Laravel project:

컴포저-Composer를 사용하여 라라벨 프로젝트에 Horizon을 설치 합니다:

    composer require laravel/horizon

After installing Horizon, publish its assets using the `horizon:install` Artisan command:

Horizon을 설치 한 뒤에 `horizon:install` 아티즌 명령어를 이용하여 에셋(assets) 파일을 퍼블리싱 합니다:

    php artisan horizon:install

You should also create the `failed_jobs` table which Laravel will use to store any [failed queue jobs](/docs/{{version}}/queues#dealing-with-failed-jobs):

[실패한 queue-큐 작업](/docs/{{version}}/queues#dealing-with-failed-jobs)을 저장하기 위한 `failed_job` 테이블을 생성해야합니다:

    php artisan queue:failed-table

    php artisan migrate

<a name="configuration"></a>
### Configuration
### 설정하기

After publishing Horizon's assets, its primary configuration file will be located at `config/horizon.php`. This configuration file allows you to configure your worker options and each configuration option includes a description of its purpose, so be sure to thoroughly explore this file.

퍼블리싱을 완료하고 나면 `config/horizon.php`라는 주요 설정파일이 복사됩니다. 이 파일을 통해서 worker의 옵션을 설정 할 수 있습니다. 각각 설정 옵션에는 용도에 대한 설명이 주석으로 표시 되어있으므로 내용을 자세히 확인 하시기 바랍니다.

#### Balance Options
#### 밸런스 옵션

Horizon allows you to choose from three balancing strategies: `simple`, `auto`, and `false`. The `simple` strategy, which is the configuration file's default, splits incoming jobs evenly between processes:

Horizon은 `simple`, `auto`, `false` 세가지 밸런싱 방법을 선택 할 수 있습니다. 환경설정 파일의 기본 값인 `simple`은 요청되는 job을 프로세스간에 균등하게 나눕니다:

    'balance' => 'simple',

The `auto` strategy adjusts the number of worker processes per queue based on the current workload of the queue. For example, if your `notifications` queue has 1,000 waiting jobs while your `render` queue is empty, Horizon will allocate more workers to your `notifications` queue until it is empty. When the `balance` option is set to `false`, the default Laravel behavior will be used, which processes queues in the order they are listed in your configuration.

`auto`는 queue의 현재 작업 부하량을 기준으로 queue당 worker 프로세스를 조절 합니다. 예를들어 `notifications` queue에 1000개의 작업이 대기중인데, `render` queue는 비어있는 경우라면, Horizon은 `notifications` queue가 비게 될때까지 더 많은 worker를 notification queue에 배정 합니다. 밸런스 옵션이 `false`일 경우에는, 라라벨 기본 동작으로 설정에 나열된 순서 대로 queue를 처리합니다.

#### Job Trimming
#### 작업 트리밍


The `horizon` configuration file allows you to configure how long recent and failed jobs should be persisted (in minutes). By default, recent jobs are kept for one hour while failed jobs are kept for a week:

`horizon` 설정 파일은 최근 작업과 실패한 작업이 유지되는 시간 (분 단위)을 설정할 수 있습니다. 기본적으로 최근 작업은 1 시간 동안 유지되고 실패한 작업은 1 주일 동안 유지됩니다.

    'trim' => [
        'recent' => 60,
        'failed' => 10080,
    ],

<a name="dashboard-authorization"></a>
### Dashboard Authorization
### Dashboard 권한 부여

Horizon exposes a dashboard at `/horizon`. By default, you will only be able to access this dashboard in the `local` environment. Within your `app/Providers/HorizonServiceProvider.php` file, there is a `gate` method. This authorization gate controls access to Horizon in **non-local** environments. You are free to modify this gate as needed to restrict access to your Horizon installation:

Horizon Dashboard는 `/horizon`으로 접속 가능하며, 기본적으로 `local` 환경에서만 접근이 가능합니다. `app/Providers/HorizonServiceProvider.php` 파일 내에 `gate` 메소드가 있습니다. 이 인증 게이트는 **비 로컬** 환경에서 Horizon에 대한 액세스를 제어합니다. Horizon 대한 액세스를 제한하기 위해 필요에 따라 이 게이트를 자유롭게 수정할 수 있습니다.

    /**
     * Register the Horizon gate.
     *
     * This gate determines who can access Horizon in non-local environments.
     *
     * @return void
     */
    protected function gate()
    {
        Gate::define('viewHorizon', function ($user) {
            return in_array($user->email, [
                'taylor@laravel.com',
            ]);
        });
    }

<a name="running-horizon"></a>
## Running Horizon
## Horizon 실행하기

Once you have configured your workers in the `config/horizon.php` configuration file, you may start Horizon using the `horizon` Artisan command. This single command will start all of your configured workers:

`config/horizon.php` 파일에서 worker의 설정을 구성한 뒤에 `horizon` 아티즌 명령어를 사용하여 horizon를 시작 할 수 있습니다. 이 하나의 명령어를 실행하면 모든 worker들이 시작됩니다:

    php artisan horizon

You may pause the Horizon process and instruct it to continue processing jobs using the `horizon:pause` and `horizon:continue` Artisan commands:

`horizon:pause` 와 `horizon:continue` 아티즌 명령어를 사용하여 Horizon 프로세스를 일시 정지하고 계속 진행 시킬 수 있습니다:

    php artisan horizon:pause

    php artisan horizon:continue

You may gracefully terminate the master Horizon process on your machine using the `horizon:terminate` Artisan command. Any jobs that Horizon is currently processing will be completed and then Horizon will exit:

`horizon:terminate` 아티즌 명령어를 사용하여 마스터 Horizon 프로세스를 안전하게(gracefully) 종료 할 수 있습니다. 이렇게 하면 Horizon이 현재 처리 중인 모든 작업이 완료되나서 프로세스가 종료됩니다:

    php artisan horizon:terminate

<a name="deploying-horizon"></a>
### Deploying Horizon
### Horizon 배포하기

If you are deploying Horizon to a live server, you should configure a process monitor to monitor the `php artisan horizon` command and restart it if it quits unexpectedly. When deploying fresh code to your server, you will need to instruct the master Horizon process to terminate so it can be restarted by your process monitor and receive your code changes.

라이브서버에 Horizon을 배포하는 경우 `php artisan horizon` 커맨드가 계속 실행되는지 프로세스 모니터를 구성하여 예기치 않게 종료되면 다시 시작해야합니다. 서버에 새로운 코드를 배포하는 경우 변경된 새로운 코드를 받은 뒤에, 프로세스 모니터가 다시 프로세스를 실행 할 수 있도록 메인 Horizon 프로세스를 종료해야합니다.

#### Supervisor Configuration
#### Supervisor 설정하기

If you are using the Supervisor process monitor to manage your `horizon` process, the following configuration file should suffice:

`horizon` 프로세스를 감시하는 프로세스 모니터로 Supervisor를 사용 하려는 경우, 아래의 설정 파일을 사용하면 충분합니다:

    [program:horizon]
    process_name=%(program_name)s
    command=php /home/forge/app.com/artisan horizon
    autostart=true
    autorestart=true
    user=forge
    redirect_stderr=true
    stdout_logfile=/home/forge/app.com/horizon.log

> {tip} If you are uncomfortable managing your own servers, consider using [Laravel Forge](https://forge.laravel.com). Forge provisions PHP 7+ servers with everything you need to run modern, robust Laravel applications with Horizon.

> {tip} 직접 서버를 관리하는게 힘들다면, [라라벨 Forge](https://forge.laravel.com) 사용을 고려 해보십시오. Forge는 Horizon을 통해 모던하고 강력한 라라벨 애플리케이션을 실행하는 데 필요한 모든 기능을 갖춘 PHP7+ 서버를 제공합니다.

<a name="tags"></a>
## Tags
## 태그

Horizon allows you to assign “tags” to jobs, including mailables, event broadcasts, notifications, and queued event listeners. In fact, Horizon will intelligently and automatically tag most jobs depending on the Eloquent models that are attached to the job. For example, take a look at the following job:

Horizon을 사용하면 mailables, event broadcasts, notifications 및 queued event listeners 작업에 "태그"를 할당할 수 있습니다. 실제로 Horizon은 job에 연결된 Eloguent 모델에 따라 대부분의 job을 지능적이고 자동으로 태그 처리합니다. 예를 들어 아래의 작업을 살펴 봅시다:

    <?php

    namespace App\Jobs;

    use App\Video;
    use Illuminate\Bus\Queueable;
    use Illuminate\Queue\SerializesModels;
    use Illuminate\Queue\InteractsWithQueue;
    use Illuminate\Contracts\Queue\ShouldQueue;
    use Illuminate\Foundation\Bus\Dispatchable;

    class RenderVideo implements ShouldQueue
    {
        use Dispatchable, InteractsWithQueue, Queueable, SerializesModels;

        /**
         * The video instance.
         *
         * @var \App\Video
         */
        public $video;

        /**
         * Create a new job instance.
         *
         * @param  \App\Video  $video
         * @return void
         */
        public function __construct(Video $video)
        {
            $this->video = $video;
        }

        /**
         * Execute the job.
         *
         * @return void
         */
        public function handle()
        {
            //
        }
    }

If this job is queued with an `App\Video` instance that has an `id` of `1`, it will automatically receive the tag `App\Video:1`. This is because Horizon will examine the job's properties for any Eloquent models. If Eloquent models are found, Horizon will intelligently tag the job using the model's class name and primary key:

`id`가 `1`인 `App\Video`인스턴스가 queue에 있는 job에 있는 경우 자동으로 `App\Video:1` 태그를 할당 받게됩니다. Horizon은 모든 Eloquent 모델의 job 속성을 확인하기 때문입니다. Eloquent 모델이 발견되면 Horizon은 모델의 클래스 이름과 기본 키를 사용하여 job에 알아서 태그를 지정합니다:

    $video = App\Video::find(1);

    App\Jobs\RenderVideo::dispatch($video);

#### Manually Tagging
#### 수동으로 태그 지정하기

If you would like to manually define the tags for one of your queueable objects, you may define a `tags` method on the class:

queueable objects에 수동으로 태그를 정하고 싶은 경우 클래스의 `tags` 메서드를 정의 하면 됩니다:

    class RenderVideo implements ShouldQueue
    {
        /**
         * Get the tags that should be assigned to the job.
         *
         * @return array
         */
        public function tags()
        {
            return ['render', 'video:'.$this->video->id];
        }
    }

<a name="notifications"></a>
## Notifications
## 알림

> **Note:** When configuring Horizon to send Slack or SMS notifications, you should review the [prerequisites for the relevant notification driver](https://laravel.com/docs/{{version}}/notifications).

> **Note:** Horizon 을 사용해서 슬랙이나 SMS 알림을 보내도록 설정할 때에는, [알림 드라이버와 연결된 사전 준비사항](https://laravel.com/docs/{{version}}/notifications)을 확인해야 합니다.

If you would like to be notified when one of your queues has a long wait time, you may use the `Horizon::routeMailNotificationsTo`, `Horizon::routeSlackNotificationsTo`, and `Horizon::routeSmsNotificationsTo` methods. You may call these methods from your application's `HorizonServiceProvider`:

대기 시간이 긴 Queue에 대한 알림을 받으려면 애플리케이션의 `AppServiceProvider`에서 `Horizon::routeMailNotificationsTo`, `Horizon::routeSlackNotificationsTo`, 또는 `Horizon::routeSmsNotificationsTo` 메서드를 사용 하십시오:

    Horizon::routeMailNotificationsTo('example@example.com');
    Horizon::routeSlackNotificationsTo('slack-webhook-url', '#channel');
    Horizon::routeSmsNotificationsTo('15556667777');

#### Configuring Notification Wait Time Thresholds
#### 알림 대기 시간의 임계값 설정하기

You may configure how many seconds are considered a "long wait" within your `config/horizon.php` configuration file. The `waits` configuration option within this file allows you to control the long wait threshold for each connection / queue combination:

`config/horizon.php` 설정 파일에 "긴 대기시간"으로 간주하는 기준 시간을 설정 할 수 있습니다. 파일 내의 `waits` 설정 옵션에서 각 연결/queue 조합에 대한 긴 대기 임계값을 제어 할 수 있습니다 :

    'waits' => [
        'redis:default' => 60,
    ],

<a name="metrics"></a>
## Metrics
## 메트릭

Horizon includes a metrics dashboard which provides information on your job and queue wait times and throughput. In order to populate this dashboard, you should configure Horizon's `snapshot` Artisan command to run every five minutes via your application's [scheduler](/docs/{{version}}/scheduling):

Horizon에는 job 및 queue의 대기 시간과 처리량에 대한 정보를 제공하는 메트릭 dashboard가 포함되어 있습니다. 이 dashboard에 정보를 제공하려면 애플리케이션의 [scheduler](/docs/{{version}}/scheduling)를 통해 Horizon의 `snapshot` 아티즌 커맨드를 5분마다 실행 하도록 설정해야 합니다:

    /**
     * Define the application's command schedule.
     *
     * @param  \Illuminate\Console\Scheduling\Schedule  $schedule
     * @return void
     */
    protected function schedule(Schedule $schedule)
    {
        $schedule->command('horizon:snapshot')->everyFiveMinutes();
    }