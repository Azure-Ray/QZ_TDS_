const padZero = num => num < 10 ? '0' + num : num;

/**
 * Do one process before saving scheduler time, because DB is GMT time
 * @param scheduler
 * @param timezone
 * @returns {string}
 */
export const saveSchedulerByGMT = (scheduler, timezone) => {
  if (!scheduler || !timezone) {
    return;
  }

  if (scheduler.startsWith('DAILY_AT:')) {
    const times = parseTimeList(scheduler);
    const processedTimes = times.map(time => revertTimeToGMT(time, timezone)).join(',');
    return formatScheduler({ type: 'DAILY_AT', processedTimes });
  } else if (scheduler.startsWith('WEEK_AT:')) {
    const times = parseTimeList(scheduler);
    const processedTimes = times.map(timeEntry => {
      const [day, time] = timeEntry.split('-').map(part => part.trim());
      const { adjustedTime, adjustedDay } = revertTimeToGMTWithDayAdjustment(time, day, timezone);
      return `${adjustedDay}-${adjustedTime}`;
    }).join(',');
    return formatScheduler({ type: 'WEEK_AT', processedTimes });
  }
};

const revertTimeToGMTWithDayAdjustment = (time, day, timezone) => {
  let [hours, minutes] = time.split(':').map(Number);
  const date = new Date();
  date.setHours(hours, minutes, 0, 0);

  if (timezone === 'Asia/Shanghai') {
    date.setHours(date.getHours() - 8);
    if (date.getUTCHours() < 0) {
      date.setUTCDate(date.getUTCDate() - 1);
      date.setUTCHours(24 + date.getUTCHours());
      day = getPreviousDay(day);
    }
  }

  return {
    adjustedTime: `${padZero(date.getUTCHours())}:${padZero(date.getUTCMinutes())}`,
    adjustedDay: day
  };
};

const getPreviousDay = (day) => {
  const daysOfWeek = ['SUN', 'MON', 'TUE', 'WED', 'THU', 'FRI', 'SAT'];
  const index = daysOfWeek.indexOf(day);
  return daysOfWeek[(index - 1 + daysOfWeek.length) % daysOfWeek.length];
};

const parseTimeList = scheduler => {
  const times = scheduler.substring(scheduler.indexOf(':') + 1).trim();
  if (times.startsWith('[') && times.endsWith(']')) {
    return times.substring(1, times.length - 1).split(',').map(t => t.trim());
  } else {
    return [times];
  }
};

const formatScheduler = ({ type, processedTimes }) => `${type}:${processedTimes}`;
