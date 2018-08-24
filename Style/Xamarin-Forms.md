# Red Pepper Xamarin.Forms Styleguide
This document outlines the general guidelines for working with Xamarin.Forms at Red Pepper.

## Contents

- [Best Application Check](#best-application-check)
- [Reactive](#reactive-programing)
- [Memory Leaks](#memory-leaks)

## Best Application Check
Do Not Use Xamarin.Forms.
If you've continued to read, this is a check to make sure you are making the appropriate decission to by pass Native Applications in Android and Ios to build them in Xamarin.Forms.  This test should default to NO, unless adaquate evidence can prove it is more appropirate than two Native Apps.  Xamarin.Forms utilizes Cross Platform Application Development, and such is not a good enough reason to use Xamarin.Forms as it does not reduce development time, and especially does not reduce debugging time.  Xamarin.Forms does not have as much documentation and support as both native platforms.  When speciallizing features in Xamarin.Forms such as GPS Location, seperate files need to be developed in respective Android and Ios enviorments.  For various other reasons in addition to those mentioned, Xamarin.Forms is not a straight forward choice for the best choice in Mobile App Development Solutions.

## Reactive Programming
Stick to Subscriptions and statements that can be debugged rather than intrinsict bindings.  Intrinsict bindings are prone to NullPointerExceptions if a ViewModel is updated buy a View has been destroyed.

_Bad_
```Xamarin
this.OneWayBind(ViewModel, vm => vm.TimerRunningTotal, v => v.RunningTotalLabel.Text);
```
_Good_
```Xamarin

this.WhenAnyValue(view => view.ViewModel.TimerRunningTotal)
    .ObserveOn(RxApp.MainThreadScheduler)
    .Subscribe(timerRunningTotal =>
    {
        this.RunningTotalLabel.Text = timerRunningTotal;
    });
```

## Memory Leaks
Do not recreate dynamic views.  Have checks, and update existing views.  

_Bad_
```Xamarin
this.WhenAnyValue(vm => vm.LastActiveEntry)
    .ObserveOn(RxApp.MainThreadScheduler)
    .Subscribe(timerEntry =>
    {
        if (timerEntry != null)
        {
            PreviousTimerControlViewModel = new TimerControlViewModel(timerEntry.StartTime, timerEntry.StopTime);
            PreviousTimerControlViewModel.HideInput = DisableInput;
        }
    });
```

_Good_
```Xamarin
this.WhenAnyValue(vm => vm.LastActiveEntry)
    .ObserveOn(RxApp.MainThreadScheduler)
    .Subscribe(timerEntry =>
    {
        if (timerEntry != null)
        {
            if (PreviousTimerControlViewModel != null)
            {
                PreviousTimerControlViewModel.UpdateViewModel(
                    timerEntry.StartTime,
                    timerEntry.StopTime);

                return;
            }

            PreviousTimerControlViewModel = new TimerControlViewModel(timerEntry.StartTime, timerEntry.StopTime);
            PreviousTimerControlViewModel.HideInput = DisableInput;
        }
    });
```