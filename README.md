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
      return `${day}-${revertTimeToGMT(time, timezone)}`;
    }).join(',');
    return formatScheduler({ type: 'WEEK_AT', processedTimes });
  }
};

const revertTimeToGMT = (time, timezone) => {
  let [hours, minutes] = time.split(':').map(Number);
  const date = new Date();
  date.setHours(hours, minutes, 0, 0);

  if (timezone === 'Asia/Shanghai') {
    date.setHours(date.getHours() - 8);
  } else {
    // Handle other timezones if necessary
  }

  // Adjust for edge cases crossing day boundaries
  if (date.getUTCHours() < 0) {
    date.setUTCDate(date.getUTCDate() - 1);
    date.setUTCHours(24 + date.getUTCHours());
  } else if (date.getUTCHours() >= 24) {
    date.setUTCDate(date.getUTCDate() + 1);
    date.setUTCHours(date.getUTCHours() - 24);
  }

  return `${padZero(date.getUTCHours())}:${padZero(date.getUTCMinutes())}`;
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
