    adjustBoundaryForShanghaiTime(date);
const adjustBoundaryForShanghaiTime = (date) => {
  if (date.getHours() < 0) {
    date.setUTCDate(date.getUTCDate() - 1);
    date.setUTCHours(24 + date.getUTCHours());
  } else if (date.getHours() >= 24) {
    date.setUTCDate(date.getUTCDate() + 1);
    date.setUTCHours(date.getUTCHours() - 24);
  }
};
