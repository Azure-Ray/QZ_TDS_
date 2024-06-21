export const saveSchedulerByGMT = (scheduler, timezone) => {
    if (!scheduler || !timezone) {
        return;
    }

    const parseTimeList = (scheduler) => {
        // 假设parseTimeList方法已经定义
        return scheduler.split(',').map(time => time.trim());
    };

    const revertTimeToGMT = (time, timezone) => {
        const [hours, minutes] = time.split(':').map(Number);
        const date = new Date();
        date.setHours(hours, minutes, 0, 0);
        
        if (timezone === 'Asia/Shanghai') {
            date.setHours(date.getHours() - 8);
        }
        
        return `${padZero(date.getUTCHours())}:${padZero(date.getUTCMinutes())}`;
    };

    const padZero = (num) => num.toString().padStart(2, '0');

    const formatScheduler = (type, processedTimes) => {
        return `${type}:${processedTimes}`;
    };

    if (scheduler.startsWith('DAILY_AT:')) {
        const times = parseTimeList(scheduler.slice(9)); // 去除 'DAILY_AT:' 部分
        const processedTimes = times.map(time => revertTimeToGMT(time, timezone)).join(', ');
        return formatScheduler('DAILY_AT', processedTimes);
    } else if (scheduler.startsWith('WEEKLY_AT:')) {
        const times = parseTimeList(scheduler.slice(10)); // 去除 'WEEKLY_AT:' 部分
        const processedTimes = times.map(timeEntry => {
            const [day, time] = timeEntry.split('-').map(part => part.trim());
            return `${day}-${revertTimeToGMT(time, timezone)}`;
        }).join(', ');
        return formatScheduler('WEEKLY_AT', processedTimes);
    }
};

// Helper function to pad numbers with leading zeros
const padZero = (num) => num.toString().padStart(2, '0');
