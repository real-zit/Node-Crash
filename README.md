# Node-Crash
Handle Exception in nodeJs application


function terminate (server, options = { coredump: false, timeout: 500 }) {
  // Exit function
  const exit = code => {
    options.coredump ? process.abort() : process.exit(code)
  }

  return (code, reason) => (err, promise) => {
    if (err && err instanceof Error) {
    // Log error information, use a proper logging library here :)
    console.log(err.message, err.stack)
    }

    // Attempt a graceful shutdown
    server.close(exit)
    setTimeout(exit, options.timeout).unref()
  }
}

module.exports = terminate

const http = require('http')
const terminate = require('./terminate')
const server = http.createServer(...)

const exitHandler = terminate(server, {
  coredump: false,
  timeout: 500
})

process.on('uncaughtException', exitHandler(1, 'Unexpected Error'))
process.on('unhandledRejection', exitHandler(1, 'Unhandled Promise'))
process.on('SIGTERM', exitHandler(0, 'SIGTERM'))
process.on('SIGINT', exitHandler(0, 'SIGINT'))
