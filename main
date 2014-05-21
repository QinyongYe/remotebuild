package main

import (
	"os"
	. "code.google.com/p/go.crypto/ssh"
	"io/ioutil"
	"github.com/spf13/cobra"
)

var view = "~/qye_unix_view0/"
var projects = []string{"Common/Search/FacebookSearchProvider"}
var machines = []string{"adric", "solaris-build1", "hpbuild1"}

func main() {
	var cmdBuild = &cobra.Command{
		Use:   "build",
		Short: "build projects",
		Run: func(cmd *cobra.Command, args []string) {
			login(run(buildCmd))
		},
	}

	var cmdClean = &cobra.Command{
		Use:   "echo",
		Short: "clean projects",
		Run: func(cmd *cobra.Command, args []string) {
			login(run(cleanCmd))
		},
	}

	var rootCmd = &cobra.Command{Use: "app"}
	rootCmd.AddCommand(cmdBuild, cmdClean)
	rootCmd.Execute()
}

func run(cb func(string) []string) func(*Client) {
	return func(c *Client) {
		for _, p := range projects {
			// Each ClientConn can support multiple interactive sessions,
			// represented by a Session.
			s, err := c.NewSession()
			if err != nil {
				panic("Failed to create session: " + err.Error())
			}
			defer s.Close()

			s.Stdout = os.Stdout
			var cmds = cb(p)
			var cmd = ""
			for _, v := range cmds {
				cmd += v + ";"
			}
			s.Run(cmd)
		}
	}
}

var cleanCmd = func (project string) []string {
	return []string{
				"cd " + view + project, // cd project folder
				view + "BuildScripts/one.pl -one -notest", // one.pl
				"make -f Makefile`uname` clean", // make
			}
}


var buildCmd = func (project string) []string {
	return []string{
		"cd " + view + project, // cd project folder
		view + "BuildScripts/one.pl -one -notest", // one.pl
		"make -j 4 -f Makefile`uname`", // make
	}
}

func login(cb func (*Client)) {
	// An SSH client is represented with a ClientConn. Currently only
	// the "password" authentication method is supported.
	//
	// To authenticate with the remote server you must pass at least one
	// implementation of AuthMethod via the Auth field in ClientConfig.
	keyfile, err := os.OpenFile("C:\\Users\\qye\\.ssh\\id_rsa", os.O_RDONLY, os.FileMode(0))
	if err != nil {
		panic("Failed to open file: " + err.Error())
	}
	defer keyfile.Close()

	keybytes, err := ioutil.ReadAll(keyfile)
	key, err := ParsePrivateKey(keybytes)
	if err != nil {
		panic("Failed to parse key: " + err.Error())
	}

	config := &ClientConfig{
		User: "qye",
		Auth: []AuthMethod{
			PublicKeys(key),
		},
	}
	client, err := Dial("tcp", "adric:22", config)
	if err != nil {
		panic("Failed to dial: " + err.Error())
	}

	cb(client)
}

