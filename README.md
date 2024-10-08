# golang-with-gocron-sample

This repository is for implementation cronjob with go example

## dependency

[gocron](https://github.com/go-co-op/gocron)
```shell=
go get github.com/go-co-op/gocron/v2
```

## create a simple interval job

```golang
func main() {
  // create a scheduler
	s, err := gocron.NewScheduler()
	if err != nil {
		log.Fatal(err)
	}

	// add a simple job to the scheduler
	j, err := s.NewJob(
		gocron.DurationJob(
			30*time.Second,
		),
		gocron.NewTask(
			func(a string) {
				log.Println(a)
			},
			"Every 30 second",
		),
		gocron.WithName("job: Every 30 seconds"),
	)
	if err != nil {
		// handle error
	}
	log.Println(j.ID())
	// Start the scheduler
	s.Start()
	// Setup a channel to listen for interrupt signals
	sigChan := make(chan os.Signal, 1)
	signal.Notify(sigChan, syscall.SIGINT, syscall.SIGTERM)
  // handle for receiving signal to stop
	go func() {
		<-sigChan
		log.Println("\nInterupt signal received, exit...")
		_ = s.Shutdown()
		os.Exit(0)
	}()

	for {

	}
}
```

## add a cronjob

```golang
// add a cronjob
	cj, err := s.NewJob(
		gocron.CronJob(
			"*/2 * * * *",
			false,
		),
		gocron.NewTask(
			func(a string) {
				log.Println(a)
			},
			"Cronjob: Every 2 mins",
		),
		gocron.WithName("Cronjob: Every 2 mins"),
	)
	log.Println(cj.ID())
```

## add daily job with 2 triggers

```golang
// add a Daily Job
	dj, err := s.NewJob(
		gocron.DailyJob(
			1, // job run every day
			gocron.NewAtTimes(
				gocron.NewAtTime(22, 11, 30),
				gocron.NewAtTime(22, 13, 30),
			),
		),
		gocron.NewTask(
			func(a string, b string) {
				log.Println(a, b)
			},
			"DailyJob",
			"Runs every day",
		),
		gocron.WithName("DailyJob: Runs every day"),
	)
	log.Println(dj.ID())
```
## add one time job that runs only once

```golang
// add one time job
	otj, err := s.NewJob(
		gocron.OneTimeJob(
			gocron.OneTimeJobStartDateTime(time.Now().Add(10*time.Second)),
		),
		gocron.NewTask(
			func(a string) {
				log.Println(a)
			},
			"OneTimeJob",
		),
		gocron.WithName("OneTimeJob"),
	)
	log.Println(otj.ID())
```

## add random duration time job

```golang
// add Random Duration Job
	drj, err := s.NewJob(
		gocron.DurationRandomJob(
			2*time.Second,
			40*time.Second,
		),
		gocron.NewTask(
			func(a string) {
				log.Println(a)
			},
			"RandomDurationJob",
		),
		gocron.WithName("RandomDurationJob"),
	)
	log.Println(drj.ID())
```
## add event listeners with job

```golang
  gocron.WithEventListeners(
    gocron.BeforeJobRuns(func(jobID uuid.UUID, jobName string) {
      log.Printf("Job staring: %s, %s \n", jobID.String(), jobName)
    }),
    gocron.AfterJobRuns(func(jobID uuid.UUID, jobName string) {
      log.Printf("Job completed: %s, %s \n", jobID.String(), jobName)
    }),
    gocron.AfterJobRunsWithError(
      func(jobID uuid.UUID, jobName string, err error) {
        log.Printf("Job has an error: %s, %s \n", jobID.String(), jobName)
      },
    ),
  ),
```