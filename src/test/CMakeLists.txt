# Copyright (c) 2018 The Bitcoin developers

project(bitcoin-test)

# Process json files.
file(MAKE_DIRECTORY "${CMAKE_CURRENT_BINARY_DIR}/data")

function(gen_json_header NAME)
	set(HEADERS "")
	foreach(f ${ARGN})
		set(h "${CMAKE_CURRENT_BINARY_DIR}/${f}.h")

		# Get the proper name for the test variable.
		get_filename_component(TEST_NAME ${f} NAME_WE)
		add_custom_command(OUTPUT ${h}
			COMMAND
				"${PYTHON_EXECUTABLE}"
				"${CMAKE_CURRENT_SOURCE_DIR}/data/generate_header.py"
				"${TEST_NAME}"
				"${CMAKE_CURRENT_SOURCE_DIR}/${f}" > ${h}
			MAIN_DEPENDENCY ${f}
			DEPENDS
				"data/generate_header.py"
			VERBATIM
		)
		list(APPEND HEADERS ${h})
	endforeach(f)
	set(${NAME} "${HEADERS}" PARENT_SCOPE)
endfunction()

gen_json_header(JSON_HEADERS
	data/base58_encode_decode.json
	data/blockfilters.json
	data/key_io_valid.json
	data/key_io_invalid.json
	data/script_tests.json
	data/sighash.json
	data/tx_invalid.json
	data/tx_valid.json
)

include(TestSuite)
create_test_suite(bitcoin)
add_dependencies(check check-bitcoin)

if(BUILD_BITCOIN_WALLET)
	set(BITCOIN_WALLET_TEST_FIXTURE
		../wallet/test/wallet_test_fixture.cpp
	)
	set(BITCOIN_WALLET_TESTS
		../wallet/test/accounting_tests.cpp
		../wallet/test/db_tests.cpp
		../wallet/test/coinselector_tests.cpp
		../wallet/test/psbt_wallet_tests.cpp
		../wallet/test/wallet_tests.cpp
		../wallet/test/walletdb_tests.cpp
		../wallet/test/wallet_crypto_tests.cpp
	)
endif()

add_boost_unit_tests_to_suite(bitcoin test_bitcoin
	jsonutil.cpp
	scriptflags.cpp
	sigutil.cpp
	test_bitcoin.cpp
	test_bitcoin_main.cpp

	# Tests generated from JSON
	${JSON_HEADERS}

	# Wallet test fixture
	${BITCOIN_WALLET_TEST_FIXTURE}

	TESTS
		activation_tests.cpp
		addrman_tests.cpp
		allocator_tests.cpp
		amount_tests.cpp
		arith_uint256_tests.cpp
		avalanche_tests.cpp
		base32_tests.cpp
		base58_tests.cpp
		base64_tests.cpp
		bip32_tests.cpp
		bitmanip_tests.cpp
		blockchain_tests.cpp
		blockcheck_tests.cpp
		blockencodings_tests.cpp
		blockfilter_tests.cpp
		blockindex_tests.cpp
		blockstatus_tests.cpp
		bloom_tests.cpp
		bswap_tests.cpp
		cashaddr_tests.cpp
		cashaddrenc_tests.cpp
		checkdatasig_tests.cpp
		checkpoints_tests.cpp
		checkqueue_tests.cpp
		coins_tests.cpp
		compress_tests.cpp
		config_tests.cpp
		core_io_tests.cpp
		crypto_tests.cpp
		cuckoocache_tests.cpp
		dbwrapper_tests.cpp
		denialofservice_tests.cpp
		descriptor_tests.cpp
		dstencode_tests.cpp
		excessiveblock_tests.cpp
		feerate_tests.cpp
		finalization_tests.cpp
		flatfile_tests.cpp
		getarg_tests.cpp
		hash_tests.cpp
		inv_tests.cpp
		key_io_tests.cpp
		key_tests.cpp
		lcg_tests.cpp
		limitedmap_tests.cpp
		mempool_tests.cpp
		merkle_tests.cpp
		merkleblock_tests.cpp
		miner_tests.cpp
		monolith_opcodes_tests.cpp
		multisig_tests.cpp
		net_tests.cpp
		netbase_tests.cpp
		pmt_tests.cpp
		policyestimator_tests.cpp
		pow_tests.cpp
		prevector_tests.cpp
		radix_tests.cpp
		raii_event_tests.cpp
		random_tests.cpp
		rcu_tests.cpp
		reverselock_tests.cpp
		rpc_tests.cpp
		rpc_server_tests.cpp
		rwcollection_tests.cpp
		sanity_tests.cpp
		scheduler_tests.cpp
		schnorr_tests.cpp
		script_bitfield_tests.cpp
		script_commitment_tests.cpp
		script_p2sh_tests.cpp
		script_standard_tests.cpp
		script_tests.cpp
		scriptnum_tests.cpp
		serialize_tests.cpp
		sigcache_tests.cpp
		sigencoding_tests.cpp
		sighash_tests.cpp
		sighashtype_tests.cpp
		sigcheckcount_tests.cpp
		sigopcount_tests.cpp
		skiplist_tests.cpp
		streams_tests.cpp
		sync_tests.cpp
		timedata_tests.cpp
		torcontrol_tests.cpp
		transaction_tests.cpp
		txindex_tests.cpp
		txvalidation_tests.cpp
		txvalidationcache_tests.cpp
		uint256_tests.cpp
		undo_tests.cpp
		util_tests.cpp
		validation_block_tests.cpp
		validation_tests.cpp
		work_comparator_tests.cpp

		# RPC Tests
		../rpc/test/server_tests.cpp

		# Wallet tests
		${BITCOIN_WALLET_TESTS}
)

target_link_libraries(test_bitcoin rpcclient server)
if(TARGET bitcoinconsensus-shared)
	target_link_libraries(test_bitcoin bitcoinconsensus-shared)
else()
	target_link_libraries(test_bitcoin bitcoinconsensus)
endif()

add_executable(test_bitcoin_fuzzy EXCLUDE_FROM_ALL
	test_bitcoin_fuzzy.cpp
)
target_link_libraries(test_bitcoin_fuzzy server)
