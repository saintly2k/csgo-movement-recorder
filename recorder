#include <regex>
#include <vector>
#include "../valve_sdk/csgostructs.hpp"
#include <iostream>
#include "../helpers/math.hpp"
#include "../options.hpp"
#include <fstream>
#include <iomanip>
#include <dirent.h>
#include <sys/stat.h>
 
namespace movementRecorder {
 
	struct custom_cmd
	{
		QAngle			viewAngles;
		Vector			pos;
		float			forwardmove;
		float			sidemove;
		float			upmove;
		int				buttons;
	};
 
	std::vector<custom_cmd> CmdFinal;
	custom_cmd tempCmd;
 
	bool recordBool;
	bool playBool;
	bool crossDistBool;
	bool AimToFirstRecord;
	bool recordAfterDonePlaying;
 
	QAngle viewPos;
	Vector lastPos;
	bool tempBool = true;
	bool isPlayingback = false;
	int i = 0;
 
	void writeVec(std::ostream& os, const std::vector<custom_cmd>& cmd)
	{
		typename std::vector<custom_cmd>::size_type size = cmd.size();
		os.write((char*)&size, sizeof(size));
		os.write((char*)&cmd[0], cmd.size() * sizeof(custom_cmd));
	}
 
	void readVec(std::istream& is, std::vector<custom_cmd>& cmd)
	{
		typename std::vector<custom_cmd>::size_type size = 0;
		is.read((char*)&size, sizeof(size));
		cmd.resize(size);
		is.read((char*)&cmd[0], cmd.size() * sizeof(custom_cmd));
	}
 
	void saveToFile()
	{
		const char* mapname = g_EngineClient->GetMapGroupName();
 
		std::string ayyy(mapname);
 
		std::string result = "customPaths/";
 
		std::regex re("\/(?!.*\/)(.*)");
 
		std::smatch match;
 
		if (std::regex_search(ayyy, match, re) && !CmdFinal.empty())
		{
			result += match.str(1);
			result += ".bin";
			std::ofstream out(result, std::ios::out | std::ios::binary);
			writeVec(out, CmdFinal);
			out.close();
		}
	}
 
	void readFromFile()
	{
		const char* mapname = g_EngineClient->GetMapGroupName();
 
		CmdFinal = {};
 
		DIR* dpdf;
		struct dirent* epdf;
		struct stat file_stats;
 
		std::string tempname = "customPaths/";
 
		dpdf = opendir("C:/Steam/steamapps/common/Counter-Strike Global Offensive/customPaths/");
 
		if (dpdf != NULL) {
			while (epdf = readdir(dpdf)) {
				if (*epdf->d_name == '.')
					continue;
 
 
				std::string temp = (std::string)epdf->d_name;
				temp = temp.substr(0, temp.size() - 4);
 
				if (strstr(mapname, temp.c_str()))
				{
					tempname += temp + ".bin";
					std::ifstream in(tempname.c_str(), std::ios::out | std::ios::binary);
					readVec(in, CmdFinal);
					in.close();
					break;
				}
			}
		}
		closedir(dpdf);
 
	}
 
	void record(CUserCmd* cmd)
	{
		if (!g_Options.movement_recording)
			return;
 
		if (g_Options.recordBool)
		{
			custom_cmd tempCmd = {};
 
			tempCmd.buttons = cmd->buttons;
			tempCmd.forwardmove = cmd->forwardmove;
			tempCmd.sidemove = cmd->sidemove;
			tempCmd.upmove = cmd->upmove;
			tempCmd.viewAngles = cmd->viewangles;
			tempCmd.pos = g_LocalPlayer->m_vecOrigin();
 
			if (tempBool)
			{
				CmdFinal = {};
			}
			CmdFinal.push_back(tempCmd);
			tempBool = false;
		}
		else
		{
			tempBool = true;
		}
	}
 
	void play(CUserCmd* cmd)
	{
		if (!g_Options.movement_recording)
			return;
 
		if (CmdFinal.size() <= 0)
			return;
 
		if (g_Options.playBool) {
			if (!isPlayingback) {
				auto dist = Math::VectorDistance(g_LocalPlayer->m_vecOrigin(), CmdFinal[0].pos);
 
				if (dist <= 1.f) {
					isPlayingback = true;
				}
				else {
					if (!crossDistBool) {
						auto viewPos = Vector(CmdFinal[0].pos[0], CmdFinal[0].pos[1], CmdFinal[0].pos[2] + 64.f);
 
						Vector temp2D;
 
						float crossDist = 0;
 
						if (Math::WorldToScreen(viewPos, temp2D)) {
							crossDist = DistanceBetweenCross(temp2D[0], temp2D[1]);
						}
						else {
							crossDist = 10000;
						}
 
						if (crossDist <= 1.f) {
							crossDistBool = true;
						}
					}
					else {
 
						//move to start point
						auto finalVec = CmdFinal[0].pos - g_LocalPlayer->m_vecOrigin();
 
						QAngle wishAngle;
						Math::VectorAngles(finalVec, wishAngle);
 
						g_EngineClient->SetViewAngles(&wishAngle);
 
						cmd->viewangles = wishAngle;
						cmd->forwardmove = dist;
 
					}
 
 
 
				}
			}
			else
			{
				if (GetAsyncKeyState(VK_LBUTTON) & 0x8000 && GetAsyncKeyState('H') & 0x8000) { //When left mouse button and H pressed we remove ending of the playing record
					while (i != CmdFinal.size()) {
						CmdFinal.pop_back();
					}
					*g_Options.recordBool = false;
					*g_Options.playBool = false;
					return;
				}
				if (GetAsyncKeyState(VK_RBUTTON) & 0x8000 && GetAsyncKeyState('H') & 0x8000) { //When right mouse button and H pressed we remove ending of the playing record and instantly start recording
					while (i != CmdFinal.size()) {
						CmdFinal.pop_back();
					}
					*g_Options.playBool = false;
					*g_Options.recordBool = true;
					tempBool = false;
					return;
				}
				if (i >= CmdFinal.size()) {
					i = 0;//reset
					isPlayingback = false;
					crossDistBool = false;
					AimToFirstRecord = true;
					if (!recordAfterDonePlaying)
						*g_Options.playBool = false;
					else {
						*g_Options.playBool = false;
						*g_Options.recordBool = true;
						tempBool = false;
					}
				}
				else
				{
 
					if (AimToFirstRecord) {
						Vector aimVec;
						Math::AngleVectors(CmdFinal[0].viewAngles, aimVec);
						Vector curVec;
						Math::AngleVectors(cmd->viewangles, curVec);
 
						auto delta = aimVec - curVec;
 
						const auto smoothed = curVec + delta / 64.f;
 
						QAngle aimAng;
						Math::VectorAngles(smoothed, aimAng);
 
						Math::Normalize3(aimAng);
						Math::ClampAngles(aimAng);
 
						g_EngineClient->SetViewAngles(&aimAng);
 
						cmd->viewangles = aimAng;
 
						auto deltadist = Math::VectorLength(delta);
 
 
						if (deltadist <= 0.0001f) {
							AimToFirstRecord = false;
						}
 
					}
					else {
						cmd->buttons = CmdFinal[i].buttons;
						cmd->forwardmove = CmdFinal[i].forwardmove;
						cmd->sidemove = CmdFinal[i].sidemove;
						cmd->upmove = CmdFinal[i].upmove;
						cmd->viewangles = CmdFinal[i].viewAngles;
 
						g_EngineClient->SetViewAngles(&CmdFinal[i].viewAngles);
 
						i++;
					}
 
				}
			}
		}
		else
		{
			i = 0;//reset
			isPlayingback = false;//stop playback
			crossDistBool = false;
			AimToFirstRecord = true;
 
		}
	}
 
	float DistanceBetweenCross(float X, float Y)
	{
		float ydist = (Y - GetSystemMetrics(SM_CYSCREEN) / 2);
		float xdist = (X - GetSystemMetrics(SM_CXSCREEN) / 2);
		float Hypotenuse = sqrt(pow(ydist, 2) + pow(xdist, 2));
		return Hypotenuse;
	}
 
}
