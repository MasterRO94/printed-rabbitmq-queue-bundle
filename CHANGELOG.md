# Change Log
All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](http://keepachangelog.com/)
and this project adheres to [Semantic Versioning](http://semver.org/).

## [Unreleased]

## [6.0.0] - 2023-03-15
### Changed
- Updated dependencies and update code to work with Symfony ^6.0 and PHP ^8.1.

### Breaking changes
- PHP >= 8.1
- Symfony ^6.0

## [5.1.0] - 2023-03-14
### Fixed
- Fix supplying dedicated entity manager to the abstract queue consumer not actually working due to the QueueTask
  itself being retrieved using the main application's entity manager.

### Changed
- Redo how one can supply a dedicated entity manager to the abstract queue consumer.

### Breaking changes
- The protected field `AbstractQueueConsumer::$repository` was removed. It's unlikely that it was used in a subclass.
- The `application_doctrine_entity_manager__service_name` bundle configuration option was removed. Please check the readme
  on how one can supply a dedicated entity manager to the abstract queue consumer now. It's unlikely that this option was used
  due to it not actually working, as mentioned in the "Fixed" section above.

## [5.0.1] - 2020-04-03
### Added
- The MakeServicesPublicAsALastResortCompilerPass that allows to make some of the
  services public again, after they have been made private in v5.0.0. This is supposed
  to be a last resort solution (see the compiler pass' docblock).

## [5.0.0] - 2020-03-30
### Changed
- Make all bundle services private. Do not retrieve services directly from the service container (apart 
  from making use of the ServiceSubscriber feature).
- Allow Symfony version ^3.4|^4.0

### Breaking changes
- All bundle services are now private. It means that, if they are to be used, they either have to be injected or used via the ServiceSubscriber Symfony feature.
- Subclasses of AbstractQueueConsumer have to use the tag: "container.service_subscriber"
- `AbstractQueueConsumer::$container` has been removed. Subclasses either need to make use of the ServiceSubscriber feature
  or use other means to obtain services.
- `AbstractQueueConsumer` constructor parameters have been changed. See readme to see what's being required now.
- The bundle now uses proper configuration, rather than relying on presence of container parameters. See readme. Configuration
  hasn't changed apart from the way it's provided so it shouldn't be a challenge to migrate it. Note that dots in
  configuration keys have been replaced with double underscores.
- The "rabbitmq-queue-bundle.queue_names_prefix" option and feature has been removed. Use RabbitMQ vhosts feature instead. See 
  readme for more details.
- `AbstractQueueConsumer::getAttemptLimit()` now returns 1 instead of 10. This means that task reattempting is now disabled by default
  instead of being enabled by default.
- The deprecated `QueueTaskInterface::STATUS_*` have been removed. Use QueueTaskStatus::* instead. 

## [4.5.3] - 2019-07-04
### Added
- Add a way to dispatch a queue task with specific queue task (message) properties. This allows to make use of e.g. rabbitmq
  x-priority feature.

### Changed
- Queue Consumer now uses "notice" level instead of "debug" when logging an occurrence of special-case exit.

## [4.5.2] - 2019-03-21
### Fixed
- Added "symfony/expression-language" as a required composer dependency
- Gracefully handle the case when an error stack trace can't be json-encoded

## [4.5.1] - 2018-05-03
### Fixed
- `DispatchDelayedQueueTasksEventListener` causing circular inclusion in some setups

## [4.5.0] - 2018-04-17
### Added
- `rabbitmq-queue-bundle.minimal_runtime_in_seconds_on_consumer_exception` option

## [4.4.0]
### Added
- `QueueTaskRepository::findByQueueNameAndStatuses()`
- **[DATABASE MIGRATION NEEDED]** Database indices on `queue_task.status` and `queue_task.queue_name`
- Sql file with the necessary schema migrations: `src/Printed/Bundle/Queue/Resources/database_migrations/version-4.4.0.sql`

## [4.3.0]
### Added
- "Late" queue payload construction capability when using `QueueTaskDispatcher::dispatchAfterNextEntityManagerFlush()`
- PreQueueTaskDispatchFn: Ability to execute a piece of code after a QueueTask is created (and flushed) but immediately
  before it's sent to the queue server.
- [minor breaking change] `ScheduledQueueTask::__construct()`'s argument list has changed 

## [4.2.1]
### Fixed
- `onTaskCancelled` lifecycle event not being called, when the task arrives to the consumer as cancelled.

## [4.2.0]
### Added
- Task lifecycle events: `onTaskCancelled` and `onTaskAbortedByException` 

## [4.1.1] - 2017-11-24
### Fixed
- `Class Printed\\Bundle\\Queue\\Entity\\QueueTask has no field or association named completion_percentage`

## [4.1.0] - 2017-11-24
### Added
- A way to cancel tasks.
- A way to express tasks' completion progress (percentage) during consumer's run.
- A way to dispatch tasks on next doctrine flush event. This is helpful to fight race condition
  between database's flush and rabbitmq's consumer start
- A couple of very dedicated methods to QueueTaskRepository for retrieving queue tasks by
  public id, status, queue name and payload content.
  
### Changed
- **[DATABASE MIGRATION NEEDED]** Add unique db index on `queue_task.id_public`.
- **[DATABASE MIGRATION NEEDED]** Add db column `queue_task.completion_percentage`.
- **[DATABASE MIGRATION NEEDED]** Add db column `queue_task.cancellation_requested`.
- Sql file with the necessary schema migrations: `src/Printed/Bundle/Queue/Resources/database_migrations/version-4.1.0.sql`

## [4.0.1] - 2017-08-21
### Changed
- Check and react on failures when reading/setting stuff in the cache.

## [4.0.0] - 2017-06-16
### Changed
- Update "php-amqplib/rabbitmq-bundle" dependency. Don't use printedcom's fork anymore.
  To fix the breaking change, please change:
```yml
graceful_max_execution_timeout: 1800
graceful_max_execution_timeout_exit_code: 10
```
to
```yml
graceful_max_execution:
    timeout: 1800
    exit_code: 10
```
in your consumers' configuration for the rabbitmq bundle, if you were using this feature. 

## [3.2.1] - 2017-05-12
### Fixed
- Fix doctrine entities being cached between consumers' runs by clearing the entity manager before
  consumers are run

## [3.2.0] - 2017-05-02
### Added
- New Deployments Detection feature, which causes workers to exit, if they run code from a previous
  project deployment

## [3.1.1] - 2017-04-24
### Changed
- Fix `console.exception` listeners unaware of fatal queue exceptions by actually throwing these exceptions.
The change here is that now consumers will crash instead of continue to the next task
when QueueFatalErrorException is thrown. This shouldn't be a breaking change, because
the consumer should be respawned by something like `supervisord`, as per all other
exceptions.

### Fixed
- Fix trying to read the exchange name in exchange-less usage.
- Fix being unable to run `queue:maintenance:wait` before db migrations, part 2.

## [3.1.0] - 2017-04-21
### Added
- Cli command for requeuing tasks. 

### Fixed
- Being unable to run `queue:maintenance:wait` before db migrations.

## [3.0.0] - 2017-04-20
### Added
- Queue maintenance mode based on cache solution instead of the filesystem.
- [breaking change] New required configuration key: `rabbitmq-queue-bundle.queue_maintenance_strategy.service_name`
- [breaking change] New required configuration key: `rabbitmq-queue-bundle.cache_queue_maintenance_strategy.cache_service_name`

## [2.3.0] - 2017-01-30
### Added
- [Minor breaking change] Add "Ensure vhost exists" command. Requires more configuration items.

## [2.2.0] - 2017-01-27
### Changed
- Log `QueueFatalErrorException`'s child exception in the database if `QueueFatalErrorException` was thrown with a child exception.

## [2.1.0] - 2017-01-27
### Added
- [Minor breaking change] Add required `rabbitmq-queue-bundle.queue_names_prefix` configuration item.

## [2.0.0] - 2017-01-23
### Changed
- [Breaking change] Use exchange-less way of using producers and consumers

[Unreleased]: https://github.com/printedcom/rabbitmq-queue-bundle/compare/5.1.0...HEAD
[5.1.0]: https://github.com/printedcom/rabbitmq-queue-bundle/compare/5.0.1...5.1.0
[5.0.1]: https://github.com/printedcom/rabbitmq-queue-bundle/compare/5.0.0...5.0.1
[5.0.0]: https://github.com/printedcom/rabbitmq-queue-bundle/compare/4.5.3...5.0.0
[4.5.3]: https://github.com/printedcom/rabbitmq-queue-bundle/compare/4.5.2...4.5.3
[4.5.2]: https://github.com/printedcom/rabbitmq-queue-bundle/compare/4.5.1...4.5.2
[4.5.1]: https://github.com/printedcom/rabbitmq-queue-bundle/compare/4.5.0...4.5.1
[4.5.0]: https://github.com/printedcom/rabbitmq-queue-bundle/compare/4.4.0...4.5.0
[4.4.0]: https://github.com/printedcom/rabbitmq-queue-bundle/compare/4.3.0...4.4.0
[4.3.0]: https://github.com/printedcom/rabbitmq-queue-bundle/compare/4.2.1...4.3.0
[4.2.1]: https://github.com/printedcom/rabbitmq-queue-bundle/compare/4.2.0...4.2.1
[4.2.0]: https://github.com/printedcom/rabbitmq-queue-bundle/compare/4.1.1...4.2.0
[4.1.1]: https://github.com/printedcom/rabbitmq-queue-bundle/compare/4.1.0...4.1.1
[4.1.0]: https://github.com/printedcom/rabbitmq-queue-bundle/compare/4.0.1...4.1.0
[4.0.1]: https://github.com/printedcom/rabbitmq-queue-bundle/compare/4.0.0...4.0.1
[4.0.0]: https://github.com/printedcom/rabbitmq-queue-bundle/compare/3.2.1...4.0.0
[3.2.1]: https://github.com/printedcom/rabbitmq-queue-bundle/compare/3.2.0...3.2.1
[3.2.0]: https://github.com/printedcom/rabbitmq-queue-bundle/compare/3.1.1...3.2.0
[3.1.1]: https://github.com/printedcom/rabbitmq-queue-bundle/compare/3.1.0...3.1.1
[3.1.0]: https://github.com/printedcom/rabbitmq-queue-bundle/compare/3.0.0...3.1.0
[3.0.0]: https://github.com/printedcom/rabbitmq-queue-bundle/compare/2.3.0...3.0.0
[2.3.0]: https://github.com/printedcom/rabbitmq-queue-bundle/compare/2.2.0...2.3.0
[2.2.0]: https://github.com/printedcom/rabbitmq-queue-bundle/compare/2.1.0...2.2.0
[2.1.0]: https://github.com/printedcom/rabbitmq-queue-bundle/compare/2.0.0...2.1.0
[2.0.0]: https://github.com/printedcom/rabbitmq-queue-bundle/compare/1.0.0...2.0.0
