package main
import (
	"fmt"
	"log"
	"net"
	"net/http"
	"github.com/grpc-ecosystem/grpc-gateway/v2/runtime"
	"google.golang.org/grpc"
	"github.com/gocql/gocql"
)
type User struct 
{
	FirstName string
	LastName  string
	Gender    string
	DOB       string
	Phone     string
	Email     string
	Blocked   bool
}
type UserServiceServer struct{}
func main() 
{
	session := connectToCassandra()
	defer session.Close()
	go startGRPCServer()
	startRESTServer()
}
func connectToCassandra() *gocql.Session 
{
	cluster := gocql.NewCluster("127.0.0.1") 
	cluster.Keyspace = "users" 
	cluster.Consistency = gocql.Quorum
	session, err := cluster.CreateSession() 
	if err != nil 
 {
		log.Fatalf("Failed to connect to Cassandra: %v", err)
	}
	fmt.Println(" Connected to Cassandra")
	return session
}
func startGRPCServer() 
{
	grpcServer := grpc.NewServer()
	listener, err := net.Listen("tcp", ":50051")
	if err != nil 
 {
		log.Fatalf("Failed to start gRPC Server: %v", err)
 }
	fmt.Println("gRPC Server is running on port 50051")
	if err := grpcServer.Serve(listener); err != nil 
 {
		log.Fatalf("gRPC Server stopped: %v", err)
	}
}
func startRESTServer() 
{
	mux := runtime.NewServeMux()
	fmt.Println("REST API running on port 8080")
	if err := http.ListenAndServe(":8080", mux); err != nil 
 {
		log.Fatalf(" REST Server failed: %v", err)
	}
}
