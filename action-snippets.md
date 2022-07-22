
Fossa runs on every PR at build time

- license scanning for dependencies to ensure compliance
- CVE scanning for dependencies to ensure active vulnerabilites are not published in artifacts
- scanning every container image that is built in CI and blocking deployment if it fails

```yml
      - name: Run FOSSA scan and upload build data
        uses: fossa-contrib/fossa-action@v1.2.0
        with:
          fossa-api-key: ${{ secrets.FOSSA_PUSH_API_KEY }}
```

codecov runs on every PR

```yml
    - uses: codecov/codecov-action@v2
      with:
        token: ${{ secrets.CODECOV_TOKEN }}
```

staticcheck runs on every PR

```yml
    - uses: dominikh/staticcheck-action@v1.0.0
      with:
        version: "2021.1.2"
        install-go: false
        cache-key: ${{ matrix.go }}
```

release cut when v0.0.1 is tagged

```yml
   - uses: ncipollo/release-action@v1
      with:
        artifacts: "winkube.tar.gz,winkube.exe"
        prerelease: true
        generateReleaseNotes: true
        commit: ${{ github.sha }}
        tag: ${{ github.ref_name }}
        bodyFile: "body.md"
        token: ${{ secrets.GITHUB_RELEASE_TOKEN }}
```

