config = {
    "s3":{
        "key": ak,
        "secret": sk,
        'anon': False,
        "client_kwargs":{'endpoint_url': end_point}
    }
}

df = dd.read_json(s3_file, lines=True, storage_options=config)