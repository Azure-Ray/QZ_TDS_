<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>SQL Version Executor</title>
    <link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/bootstrap/4.5.2/css/bootstrap.min.css">
</head>
<body>
    <div id="root"></div>
    <script src="https://unpkg.com/react/umd/react.production.min.js"></script>
    <script src="https://unpkg.com/react-dom/umd/react-dom.production.min.js"></script>
    <script src="https://unpkg.com/babel-standalone@6.26.0/babel.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/axios/dist/axios.min.js"></script>
    <script type="text/babel">

      class YourComponent extends React.Component {
        state = {
          isFetching: false,
          dbName: '',
          scriptType: '',
          jdbcList: [],
          dbJdbcLink: ''
        };

        handleDbNameClick = async () => {
          if (this.state.jdbcList.length === 0) {
            try {
              const response = await axios.get('/get-jdbc-list');
              this.setState({ jdbcList: response.data });
            } catch (error) {
              console.error("Error fetching JDBC list", error);
            }
          }
        };

        handleDbNameChange = (event) => {
          const selectedDbName = event.target.value;
          const selectedDb = this.state.jdbcList.find(db => db.name === selectedDbName);
          const selectedDbJdbcLink = selectedDb ? selectedDb.value : '';
          this.setState({ dbName: selectedDbName, dbJdbcLink: selectedDbJdbcLink });
        };

        handleScriptTypeChange = (event) => {
          this.setState({ scriptType: event.target.value });
        };

        handleModalClose = () => {
          this.setState({ isFetching: false });
          this.modal.hide(); // Ensure the modal is hidden
        };

        render() {
          const { dbName, scriptType, jdbcList, isFetching } = this.state;
          const avatarUrl = `https://photos-global.hsbc/casual/square/${getCookie("staffId").substring(0, 4)}/${getCookie("staffId")}.jpg`;

          return (
            <div className="container p-5">
              <div className="header-container d-flex justify-content-between align-items-center mb-4">
                <h1 className="header-text">Welcome, {getCookie("displayName")}</h1>
                <img src={avatarUrl} alt="User Avatar" style={{ width: '50px', height: '50px', borderRadius: '50%' }} />
              </div>
              <h1>SQL Version Executor</h1>
              <div className="mb-3">
                <label htmlFor="dbName" className="form-label">Database Name</label>
                <select
                  className="form-select"
                  id="dbName"
                  value={dbName}
                  onClick={this.handleDbNameClick}
                  onChange={this.handleDbNameChange}
                >
                  <option value="">Select Database</option>
                  {jdbcList.map(db => (
                    <option key={db.name} value={db.name}>{db.name}</option>
                  ))}
                </select>
              </div>
              <div className="mb-3">
                <label htmlFor="scriptType" className="form-label">SQL Type</label>
                <select className="form-select" id="scriptType" value={scriptType} onChange={this.handleScriptTypeChange}>
                  <option value="">Select SQL Type</option>
                  <option value="ddl">DDL</option>
                  <option value="dml">DML</option>
                </select>
              </div>
            </div>
          );
        }
      }

      const getCookie = (name) => {
        const value = `; ${document.cookie}`;
        const parts = value.split(`; ${name}=`);
        if (parts.length === 2) return parts.pop().split(';').shift();
      };

      ReactDOM.render(<YourComponent />, document.getElementById('root'));
    </script>
</body>
</html>
