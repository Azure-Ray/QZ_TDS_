import React, { Component } from 'react';
import axios from 'axios';

class YourComponent extends Component {
  state = {
    isFetching: false,
    dbName: '',
    scriptType: '',
    jdbcList: [],
    dbJdbcLink: ''
  };

  componentDidMount() {
    this.fetchJdbcList();
  }

  fetchJdbcList = async () => {
    try {
      const response = await axios.get('/get-jdbc-list');
      this.setState({ jdbcList: response.data });
    } catch (error) {
      console.error("Error fetching JDBC list", error);
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
          <select className="form-select" id="dbName" value={dbName} onChange={this.handleDbNameChange}>
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

export default YourComponent;
