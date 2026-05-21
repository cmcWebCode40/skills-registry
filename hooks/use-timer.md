---
name: use-timer
category: hooks
stack: [react, react-native, typescript]
keywords: [countdown, timer, otp, appstate, formatting]
source-files: [libs/hooks/useTimer.ts]
---

# useTimer

## Problem
You need a countdown timer hook for OTP resend delays, verification timeouts, or any timed operations that survives app backgrounding.

## When to Use
- Building OTP verification flows with resend timers
- Creating countdown timers that must survive app backgrounding
- Implementing time-locked features
- Formatting time display as "mins : secs"

## Implementation

### Code

```typescript
import { useCallback, useEffect, useState, useRef } from 'react';
import { AppState } from 'react-native';

interface UseTimerReturn {
  timeLeft: string; // Formatted as "0mins : 30secs"
  hasTimeout: boolean;
  resetOtpTimer: () => void;
}

export const useTimer = (seconds: number): UseTimerReturn => {
  const [timeLeft, setTimeLeft] = useState(seconds);
  const [hasTimeout, setHasTimeout] = useState(false);
  const startTimeRef = useRef<number>(Date.now());
  const timerRef = useRef<NodeJS.Timeout | null>(null);

  const formatTime = useCallback((time: number): string => {
    const minutes = Math.floor(time / 60);
    const secs = time % 60;
    return `${minutes}mins : ${secs.toString().padStart(2, '0')}secs`;
  }, []);

  const clearTimer = () => {
    if (timerRef.current) {
      clearInterval(timerRef.current);
      timerRef.current = null;
    }
  };

  const startTimer = useCallback(() => {
    clearTimer();
    startTimeRef.current = Date.now();
    setHasTimeout(false);
    setTimeLeft(seconds);

    timerRef.current = setInterval(() => {
      const elapsedTime = Math.floor((Date.now() - startTimeRef.current) / 1000);
      const remainingTime = seconds - elapsedTime;

      if (remainingTime > 0) {
        setTimeLeft(remainingTime);
      } else {
        setTimeLeft(0);
        setHasTimeout(true);
        clearTimer();
      }
    }, 1000);
  }, [seconds]);

  const resetOtpTimer = useCallback(() => {
    startTimer();
  }, [startTimer]);

  // Start timer on mount
  useEffect(() => {
    startTimer();
    return () => {
      clearTimer();
    };
  }, [startTimer]);

  // Handle app backgrounding (pause/resume)
  useEffect(() => {
    const handleAppStateChange = (nextAppState: string) => {
      if (nextAppState === 'active') {
        // App resumed: recalculate time based on elapsed wall clock
        const elapsedTime = Math.floor((Date.now() - startTimeRef.current) / 1000);
        const remainingTime = seconds - elapsedTime;
        setTimeLeft(remainingTime > 0 ? remainingTime : 0);
      }
    };

    const subscription = AppState.addEventListener('change', handleAppStateChange);

    return () => {
      subscription.remove();
    };
  }, [seconds]);

  return {
    timeLeft: formatTime(timeLeft),
    hasTimeout,
    resetOtpTimer,
  };
};

export default useTimer;
```

## Usage Example

**OTP Verification Screen:**

```typescript
import useTimer from 'libs/hooks/useTimer';
import { Button, Paragraph } from 'components/ui';

export default function OtpVerificationScreen() {
  const { timeLeft, hasTimeout, resetOtpTimer } = useTimer(300); // 5 minutes

  const handleResend = async () => {
    // Resend OTP
    resetOtpTimer(); // Restart timer
  };

  return (
    <View>
      <Paragraph size="sm" color="secondary">
        Resend OTP in: {timeLeft}
      </Paragraph>
      <Button
        disabled={!hasTimeout}
        onPress={handleResend}
      >
        Resend OTP
      </Button>
    </View>
  );
}
```

## Gotchas

- **AppState resume**: Timer is recalculated on app resume using wall-clock time (Date.now()). This prevents timer "cheating" by force-closing the app.
- **Interval cleanup**: Always clear interval on unmount to prevent memory leaks.
- **Format output**: Formatted string includes zero-padding ("05secs" not "5secs").
- **hasTimeout**: Becomes true only when time reaches 0. Use this to enable resend button.
- **Seconds precision**: Timer updates every 1000ms (1 second). Sub-second precision not supported.
